# Index Signatures in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** An **index signature** is a TypeScript type-level construct that declares an object type to be **indexable by a family of keys** of a specified type (`string`, `number`, or `symbol`), mapping each such key to a value of a specified type. It allows types to describe objects whose property names are **not known at compile time** while still enforcing a consistent value type for all dynamically accessed properties.

**Technical Definition.** An index signature has the syntactic form `[key: KeyType]: ValueType`, where `KeyType` is `string`, `number`, or `symbol` (and, since TypeScript 4.4, template literal types). When declared in an interface or type alias, it permits bracket-notation access with keys of `KeyType`, returning `ValueType`. The compiler enforces that **explicitly declared properties are assignable to the index signature's value type**, and that indexed accesses return `ValueType` (or `ValueType | undefined` under `noUncheckedIndexedAccess`). Index signatures are **erased** during compilation and have no runtime representation. The `keyof` operator on a type with a string index signature resolves to `string | number` because JavaScript coerces numeric keys to strings.

**Beginner-Friendly Explanation.** An index signature is **a rule that says “this object can be indexed with any key of a certain kind, and every such key gives you a value of a certain type.”** Think of a dictionary: you do not know every word in advance, but you know that looking up any word gives you a definition (a string). An index signature on a type says: “You can use bracket notation with any string key on this object, and you will get back a value of this type.” It lets TypeScript check dynamic property access that would otherwise be `any` and lose all safety.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Dynamic key support** | Allows arbitrary keys of type `string`, `number`, or `symbol` (and template-literal patterns since TS 4.4). |
| **Value-type guarantee** | Every indexed access returns the declared `ValueType` (or `ValueType \| undefined` under strict index checking). |
| **Explicit-property compatibility** | All explicitly declared properties must be assignable to the index signature's value type. |
| **`keyof` expansion** | A string index signature makes `keyof T` include `string \| number`; a number index signature makes it include `number`. |
| **Runtime coercion** | JavaScript coerces numeric keys to strings, so `obj[1]` and `obj["1"]` access the same property. |
| **Erasure** | Index signatures exist only at compile time and emit no JavaScript. |
| **No required keys** | An object with zero properties satisfies an index signature; no specific key is required. |

---

### Prerequisites

- **TypeScript basics**: interfaces, type aliases, and object types.
- **Bracket notation** and the distinction between dot notation (`obj.prop`) and indexed access (`obj["prop"]`).
- **Utility types**: `Record<K, V>` as a common alternative.
- **Compiler flags**: `noUncheckedIndexedAccess` and `noPropertyAccessFromIndexSignature`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Dictionary and map typing** in TypeScript.
- **Dynamic configuration objects** and API response shapes.
- **Template literal types** (TypeScript 4.4+) for constrained key patterns.
- **`Record<K, V>` and mapped types** as more precise alternatives.
- **`Map` and `Set`** for truly dynamic runtime key-value storage.

---

## Core Concepts / Features

---

## 1. Index Signatures

### Definitions

**Core Definition.** An **index signature** is a type-level declaration of the form `[key: KeyType]: ValueType` that permits an object type to be indexed with keys of `KeyType`, returning values of `ValueType`. It enables static typing of objects whose property names are not known at compile time.

**Technical Definition.** Index signatures may appear in interfaces, type aliases, and class bodies. The key type may be `string`, `number`, or `symbol` (and template literal types since TS 4.4). When both string and number index signatures are present, the number-indexed value type must be **assignable to** the string-indexed value type, reflecting JavaScript's coercion of numeric keys to strings. Explicitly declared properties must be assignable to the index signature's value type; otherwise, the compiler reports an error. Indexed accesses return `ValueType`; under `--noUncheckedIndexedAccess`, they return `ValueType | undefined`.

**Beginner-Friendly Explanation.** An index signature is **a way to tell TypeScript “this object is a dictionary.”** You say: “Any string key is allowed, and every value is a `string`.” Then TypeScript lets you write `obj["anyKey"]` and knows the result is a `string`. It is useful when the keys are dynamic but the values follow a consistent type.

### Purposes

- **To type dynamically keyed objects** without falling back to `any`.
- **To enable safe bracket-notation access** on objects whose keys are not known in advance.
- **To model dictionaries, caches, configuration maps, and API responses** with variable keys.
- **To combine known and unknown properties** in a single type, where known properties have specific types and additional properties follow the index signature.
- **To support template-literal-constrained keys** such as `data-*` attributes or event-handler names (TS 4.4+).

### Syntax Rules and Structure

**General syntax (interface or type alias):**

```
interface TypeName {
  [key: KeyType]: ValueType;
}
```

**Component breakdown:**

- `[key` — opens the index signature; `key` is an arbitrary parameter name (can be `index`, `prop`, etc.).
- `: KeyType` — the permitted key type (`string`, `number`, `symbol`, or a template literal).
- `]: ValueType` — the value type returned for any key of `KeyType`.
- `;` — terminates the declaration.

**Combined with explicit properties:**

```
interface UserProfile {
  id: string;
  email: string;
  [key: string]: string;
}
```

- `id` and `email` are explicit required properties.
- The index signature allows additional string keys, all with `string` values.
- Explicit property types must be assignable to `ValueType`.

**Syntax rules.**

- The key type must be `string`, `number`, `symbol`, or a template literal type.
- Only **one** index signature per key type is allowed in a type.
- When both string and number signatures exist, the number value type must be a subtype of the string value type.
- Explicit properties must be assignable to the index signature's value type.
- `readonly` may be applied to index signatures.

**Constraints and limitations.**

- Index signatures assign **the same value type to every key**; they cannot give different keys different types (use mapped types or `Record` with literal unions instead).
- Index signatures **do not require any specific keys**; an empty object satisfies an index signature.
- Index signatures **do not catch typos**; `obj.nmae` compiles if `"nmae"` is a valid string key.
- Under `noUncheckedIndexedAccess`, indexed access returns `ValueType | undefined`, but by default the compiler **pretends the key exists** even when it does not.

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

#### Example 2: Combining known properties with an index signature

```typescript
// example_index_combined.ts
interface UserProfile {
  id: string;
  email: string;
  [key: string]: string;
}

const user: UserProfile = {
  id: "user-123",
  email: "user@example.com",
  department: "Engineering",
  location: "New York",
};

console.log(user.email);      // Expected: user@example.com
console.log(user.department); // Expected: Engineering
```

**Expected Output:**
```
user@example.com
Engineering
```

**Why this output occurs.** `id` and `email` are explicit required properties, and both are `string`, so they are assignable to the index signature's `string` value type. Additional keys like `department` and `location` are permitted by the index signature. This pattern lets you type objects with a fixed core and flexible extensions.

---

### Real-World Cases

- **Configuration maps**: `interface EnvConfig { [key: string]: string | undefined }` for environment variables.
- **Feature flags**: `interface FeatureFlags { [flagName: string]: boolean }` for toggles with dynamic names.
- **API response caches**: `interface ResponseCache { [endpoint: string]: { data: unknown; timestamp: number } }`.
- **CSS custom properties**: `interface ThemeVars { [key: \`--${string}\`]: string | number }`.

---

## 2. `[key: string]: T`

### Definitions

**Core Definition.** A **string index signature** declares that an object may be indexed with **any string key**, returning a value of type `T`. It is the most common form of index signature and models dictionary-like objects.

**Technical Definition.** The signature `[key: string]: T` permits bracket-notation and dot-notation access with any string (including stringified numbers). The compiler checks that all explicitly declared properties are assignable to `T`. The `keyof` operator on a type with a string index signature resolves to `string | number` because JavaScript coerces numeric keys to strings. Under `noUncheckedIndexedAccess`, accessing an undeclared key returns `T | undefined`.

**Beginner-Friendly Explanation.** A string index signature says: **“You can use any string as a key, and every key gives you a value of type `T`.”** It is the type for a dictionary where the words are strings and the definitions all have the same type.

### Purposes

- **To model dictionaries and maps** keyed by strings.
- **To allow dynamic property access** on configuration and API response objects.
- **To combine a fixed set of known properties with arbitrary additional properties** of a consistent type.
- **To enable bracket and dot notation** on objects with unknown keys.

### Syntax Rules and Structure

**General syntax:**

```
interface TypeName {
  [key: string]: T;
}
```

**Component breakdown:**

- `[key: string]` — the index signature parameter: any string key.
- `: T` — the value type for all string-keyed properties.

**Syntax rules.**

- All explicit properties must be assignable to `T`.
- The key parameter name is arbitrary; `key`, `index`, `name`, etc. are all valid.
- A type may have at most **one** string index signature.
- The signature permits both dot notation (`obj.prop`) and bracket notation (`obj["prop"]`).
- Numeric keys are permitted because they are coerced to strings.

**Constraints and limitations.**

- **No typo protection**: any string key is allowed, so `obj.nmae` compiles if `"nmae"` is a string.
- **No required keys**: an empty object satisfies the signature.
- **Same value type for all keys**: cannot give different keys different types without a union or mapped type.
- **`noUncheckedIndexedAccess` may be noisy**: every indexed access becomes `T | undefined`.

### Annotated Complete Step by Step Code Examples

#### Example 1: String dictionary

```typescript
// example_string_index.ts
interface StringDictionary {
  [key: string]: string;
}

const colors: StringDictionary = {
  primary: "blue",
  secondary: "green",
};

colors.accent = "red";
console.log(colors.primary); // Expected: blue
console.log(colors["accent"]); // Expected: red
```

**Expected Output:**
```
blue
red
```

**Why this output occurs.** The string index signature permits any string key with a string value. Both dot and bracket notation work. Adding `accent` is allowed because it is a string key with a string value.

---

#### Example 2: `noUncheckedIndexedAccess` changes the return type

```typescript
// example_string_index_unchecked.ts
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
  console.log(missing.toUpperCase());
}
```

**Expected Output:**
```
localhost
undefined
```

**Why this output occurs.** Under `noUncheckedIndexedAccess`, every indexed access returns `T | undefined`, forcing the developer to check for absence. Without this flag, `config.missing` would be typed as `string`, and calling `.toUpperCase()` would crash at runtime.

---

### Real-World Cases

- **Environment variables**: `process.env` typed as `{ [key: string]: string | undefined }`.
- **Feature flag maps**: `{ [flag: string]: boolean }` for dynamic toggles.
- **Configuration objects**: `{ [setting: string]: string | number | boolean }` for app settings.

---

## 3. `[key: number]: T`

### Definitions

**Core Definition.** A **number index signature** declares that an object may be indexed with **numeric keys**, returning a value of type `T`. It is useful for array-like structures and numeric-keyed maps.

**Technical Definition.** The signature `[index: number]: T` permits bracket-notation access with numeric keys. At runtime, JavaScript **coerces numeric keys to strings**, so `obj[1]` and `obj["1"]` access the same property. When both string and number index signatures exist, the number value type must be a **subtype** of the string value type. The `keyof` operator on a type with a number index signature includes `number`.

**Beginner-Friendly Explanation.** A number index signature says: **“You can use numbers as keys, and every number key gives you a value of type `T`.”** It is useful for objects that look like arrays but are not actual arrays—such as a map of HTTP status codes to messages.

### Purposes

- **To model array-like or numeric-keyed objects** that are not `Array` instances.
- **To type maps keyed by numeric codes** (HTTP status codes, error codes, enum values).
- **To describe the numeric index behavior of arrays** when accepting `ArrayLike<T>` structures.
- **To combine with string index signatures** for objects that accept both key types.

### Syntax Rules and Structure

**General syntax:**

```
interface TypeName {
  [index: number]: T;
}
```

**Component breakdown:**

- `[index: number]` — the index signature parameter: any number key.
- `: T` — the value type for all number-keyed properties.

**Syntax rules.**

- A type may have at most **one** number index signature.
- When combined with a string index signature, `T` (number) must be a subtype of the string signature's value type.
- The key parameter name is arbitrary.
- Numeric keys are coerced to strings at runtime.

**Constraints and limitations.**

- **`Object.keys()` returns strings**, not numbers, even for numeric keys.
- **False mental model**: numeric index signatures may mislead developers into thinking numeric keys are "real" at runtime.
- **Prefer `Array<T>` or `ArrayLike<T>`** for genuine array-like structures; numeric index signatures are a "helpful fiction" that does not reflect JavaScript's runtime behavior.
- Some style guides recommend **never** using number as an index signature type.

### Annotated Complete Step by Step Code Examples

#### Example 1: Number-keyed error messages

```typescript
// example_number_index.ts
interface ErrorMessages {
  [statusCode: number]: string;
}

const errors: ErrorMessages = {
  404: "Not Found",
  500: "Internal Server Error",
  403: "Forbidden",
};

console.log(errors[404]); // Expected: Not Found
console.log(Object.keys(errors)); // Expected: [ '403', '404', '500' ]
```

**Expected Output:**
```
Not Found
[ '403', '404', '500' ]
```

**Why this output occurs.** The number index signature allows numeric keys. At runtime, JavaScript coerces `404` to the string `"404"`, so `Object.keys()` returns strings. TypeScript tracks the keys as numbers at compile time, but the runtime reality is strings.

---

#### Example 2: Combining string and number index signatures

```typescript
// example_both_index.ts
interface MixedDictionary {
  [key: string]: string | number;
  [index: number]: string; // Must be subtype of string | number
}

const dict: MixedDictionary = {
  0: "zero",
  name: "Ada",
};

console.log(dict[0]);      // Expected: zero
console.log(dict["name"]); // Expected: Ada
```

**Expected Output:**
```
zero
Ada
```

**Why this output occurs.** The number index signature's value type (`string`) is a subtype of the string index signature's value type (`string | number`), satisfying the compatibility rule. Both numeric and string keys are permitted.

---

#### Example 3: Prefer `Array<T>` over a numeric index signature

```typescript
// example_array_preferred.ts
// ❌ Numeric index signature (not recommended for arrays).
// type Bad = { [index: number]: string };

// ✅ Array<T> is clearer and reflects runtime behavior.
type Good = string[];

const items: Good = ["a", "b", "c"];
console.log(items[0]); // Expected: a
```

**Expected Output:**
```
a
```

**Why this output occurs.** `Array<T>` correctly models arrays, including their methods and length property, and reflects the string-keyed runtime reality. Numeric index signatures are a compile-time fiction that should be avoided for genuine array-like structures.

---

### Real-World Cases

- **HTTP status code maps**: `{ [code: number]: string }` mapping codes to messages.
- **Sparse arrays or array-like objects**: `ArrayLike<T>` uses a numeric index signature internally.
- **Enum-keyed maps**: using numeric enum values as keys (though `Record` with the enum type is often clearer).

---

## 4. Index Signature Best Practices

### Definitions

**Core Definition.** **Index signature best practices** are guidelines for using index signatures in ways that maximize **type safety, clarity, and tooling effectiveness**, while avoiding the loss of type information and autocomplete that occurs when index signatures are used where more precise types would suffice.

**Technical Definition.** Best practices center on the principle articulated by *Effective TypeScript*: **“If you know the property names, don't use an index signature.”** Index signatures are imprecise by design—they allow any key of the declared type (including typos), require no specific keys, cannot assign distinct types to different keys, and defeat autocomplete. Better alternatives include **interfaces** for known properties, **`Record<K, V>`** for unions of known keys, **optional properties** for partial known sets, **union types** for precise combinations, and **`Map`** for truly dynamic runtime keys. When index signatures are appropriate—allowing additional properties or constraining key patterns—they should be **as narrow as possible**, use **specific value types instead of `any`**, and enable **`noUncheckedIndexedAccess`** to acknowledge that indexed lookups may return `undefined`.

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
8. **Prefer `Record` over bare index signatures for consistency.** TypeScript ESLint's `consistent-indexed-object-style` rule recommends choosing one style and using it consistently.
9. **Avoid numeric index signatures for array-like structures.** Use `Array<T>`, tuples, or `ArrayLike<T>` instead.

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
- `Record` is generally preferred for simple cases because it is less boilerplate and can use literal types.

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

- TSConfig Reference — Index Signature Glossary - https://www.typescriptlang.org/glossary/
- TypeScript Handbook — Index Signatures (Object Types) - https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures
- TypeScript 2.9 Release Notes — `keyof` with Numeric and Symbol Keys - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-9.html
- TSConfig Reference — `noUncheckedIndexedAccess` - https://www.typescriptlang.org/tsconfig/noUncheckedIndexedAccess.html
- TSConfig Reference — `noPropertyAccessFromIndexSignature` - https://www.typescriptlang.org/tsconfig/noPropertyAccessFromIndexSignature.html
- How to Implement Index Signatures in TypeScript — OneUptime Blog - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-30-typescript-index-signatures/README.md
- Index Signature — TypeScript Guide by Convex - https://www.convex.dev/typescript/advanced/type-operators-manipulation/typescript-index-signature
- Prefer More Precise Alternatives to Index Signatures — Effective TypeScript Skills - https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/marius-townhouse/effective-typescript-skills/index-signature-alternatives/SKILL.md
- Avoid Numeric Index Signatures — Effective TypeScript Skills - https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/marius-townhouse/effective-typescript-skills/avoid-numeric-index/SKILL.md
- TypeScript ESLint — `consistent-indexed-object-style` - https://typescript-eslint.io/rules/consistent-indexed-object-style/
- TypeScript Record Type with Examples — Refine Blog - https://refine.dev/blog/typescript-record-type
- Stack Overflow — Why is a number a valid key of `{ [key: string]: unknown }`? - https://stackoverflow.com/posts/77417989/revisions