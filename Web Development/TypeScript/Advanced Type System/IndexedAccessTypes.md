# Indexed Access Types in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Indexed access types** — also called **lookup types** — are a TypeScript type operator that **retrieves the type of a property at a specific key** from another type. They are the type-level dual of runtime element access: syntactically they look exactly like `obj[key]`, but they are written in a type position and operate on types rather than values. Indexed access types are the foundation for expressing **type-safe dynamic property access**, deriving **property and element types**, and building **mapped and utility types**.

**Technical Definition.** For a type `T` and a key type `K` that is assignable to `keyof T`, the indexed access type `T[K]` yields the type of the property at key `K`. If `K` is a **union of keys**, `T[K]` yields the **union of the corresponding property types**. If `T` contains a **numeric index signature**, `T[number]` yields the element type. Indexed access types support **nested chaining** (`T[K][J]`) and composition with `keyof`, `typeof`, conditional types, and generics. They are **erased** during compilation and emit no JavaScript.

**Beginner-Friendly Explanation.** An indexed access type is **“give me the type of this property.”** If you have a type `Person` with a `name: string` property, then `Person["name"]` is `string`. You can ask for several properties at once (`Person["name" | "age"]`) and get a union of their types. You can also use it on arrays: `string[][0]` is `string`, and `MyArray[number]` gives you the element type. It is how you **look up types by key**, just like you look up values by key in JavaScript.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Type-level lookup** | Retrieves the type of a property from another type, not a value. |
| **Union distribution** | Indexing with a union of keys produces a union of property types. |
| **Numeric indexing** | `T[number]` extracts element types from arrays, tuples, and numeric index signatures. |
| **Nested chaining** | Brackets can be chained: `T["a"]["b"]` descends into nested types. |
| **Generic-safe** | Combined with `K extends keyof T`, `T[K]` preserves precise property types in generics. |
| **Composable** | Works with `keyof`, `typeof`, mapped types, conditional types, and template literal types. |
| **Strict-mode sensitive** | Under `noUncheckedIndexedAccess`, indexed reads add `\| undefined` for potentially missing keys. |
| **Erasure** | All indexed access types are compile-time only and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type annotations, interfaces, type aliases, and unions.
- **Generics fundamentals**: type parameters and constraints (`<T extends U>`).
- **The `keyof` operator**: the dual of indexed access types.
- **Literal types**: `"name"`, `42`, and `keyof` results.
- **Mapped types** at an introductory level.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **`keyof` and lookup types** — the dual operators for key extraction and value extraction.
- **Mapped types** — use indexed access (`T[P]`) to preserve property types during transformation.
- **Utility types** — `Pick<T, K>`, `Omit<T, K>`, and `Record<K, V>` rely on indexed access and `keyof`.
- **Generic constraints** — `K extends keyof T` with `T[K]` return types.
- **`typeof` and `as const`** — derive indexed access inputs from runtime values.
- **`noUncheckedIndexedAccess`** — strict flag that adds `| undefined` to indexed reads.

---

## Core Concepts / Features

---

## 1. Indexed Access Types

### Definitions

**Core Definition.** An **indexed access type** looks up the type of a property on another type using bracket syntax: `Type["key"]`. It is the type-level equivalent of JavaScript's `object[key]` access.

**Technical Definition.** `T[K]` requires `K` to be assignable to `keyof T` (or assignable to `number` if `T` has a numeric index signature). The result is the **type of the property** at `K`, or the **union of property types** if `K` is a union. Indexed access types may be used anywhere a type is expected and are **erased** at compile time.

**Beginner-Friendly Explanation.** An indexed access type is **asking TypeScript “what type is this property?”** `Person["age"]` gives you `number`. It is a way to extract a piece of a type without redefining it.

### Purposes

- **To extract the type of a specific property** from an object type without duplicating annotations.
- **To derive types from other types** so changes to the source type propagate automatically.
- **To express type-safe dynamic property access** in generic functions.
- **To build utility types** that preserve property types during transformation.
- **To avoid manual type duplication** when the source type is the single source of truth.

### Syntax Rules and Structure

**General syntax:**

```
type PropertyType = Type["propertyName"];
```

**Component breakdown:**

- `Type` — the type being indexed.
- `["propertyName"]` — the key or union of keys to look up.
- `PropertyType` — the resulting property type (or union of property types).

**Union indexing:**

```
type T = Person["name" | "age"]; // string | number
```

**Syntax rules.**

- The key must be a **type**, not a value; `const` variables cannot be used as index keys.
- Only **bracket notation** is supported; dot notation (`T.a`) is invalid in type positions.
- Indexing a non-existent property produces a compile-time error.
- Union keys produce a union of property types.
- Indexed access types may be combined with `keyof`, `typeof`, and generics.

**Constraints and limitations.**

- Indexing a property that does not exist is an error.
- Generic indexed access may widen to the constraint in certain positions, losing precision.
- Indexed access on an optional property includes `undefined`.
- `noUncheckedIndexedAccess` adds `| undefined` to index-signature reads.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic property lookup

```typescript
// example_indexed_basic.ts
// Step 1: Define a source type.
interface Person {
  name: string;
  age: number;
  location: string;
}

// Step 2: Look up individual property types.
type NameType = Person["name"];       // string
type AgeType = Person["age"];         // number

// Step 3: Use the derived types.
const name: NameType = "Ada";
const age: AgeType = 36;

console.log(name, age); // Expected: Ada 36
```

**Setup.** Compile with `npx tsc example_indexed_basic.ts --strict`.

**Expected Output:**
```
Ada 36
```

**Why this output occurs.** `Person["name"]` resolves to `string`; `Person["age"]` resolves to `number`. The derived aliases remain synchronized with `Person` if it changes.

---

#### Example 2: Union indexing

```typescript
// example_indexed_union.ts
interface Person {
  name: string;
  age: number;
  alive: boolean;
}

// Union of keys → union of property types.
type StringOrNumber = Person["name" | "age"]; // string | number

// keyof yields the union of all keys.
type AllValues = Person[keyof Person]; // string | number | boolean

const value: StringOrNumber = 42;
console.log(value); // Expected: 42
```

**Expected Output:**
```
42
```

**Why this output occurs.** `Person["name" | "age"]` indexes `Person` with `"name" | "age"`, producing `string | number`. `Person[keyof Person]` indexes with all keys, producing the union of all property types.

---

#### Example 3: Non-existent key error

```typescript
// example_indexed_error.ts
interface Person {
  name: string;
  age: number;
}

// ❌ Typo is caught at compile time.
// type Bad = Person["nmae"];
// Error: Property 'nmae' does not exist on type 'Person'.

console.log("Non-existent key is rejected");
```

**Expected Output:**
```
Non-existent key is rejected
```

**Why this output occurs.** Indexed access types only permit keys that exist on the source type. Typos are caught before runtime.

---

### Real-World Cases

- **Deriving property types**: `type UserId = User["id"]`.
- **Union of property types**: `type SortableValue = Item["name" | "createdAt"]`.
- **Generic property access**: `function get<T, K extends keyof T>(obj: T, key: K): T[K]`.

---

## 2. `T[K]` — Generic Indexed Access

### Definitions

**Core Definition.** `T[K]` is the **generic form** of indexed access, where both the source type `T` and the key type `K` are type parameters. It is the canonical way to **preserve the precise type of a property** in generic functions.

**Technical Definition.** When `K extends keyof T`, `T[K]` is the type of the property selected by `K`. In a generic function, `T[K]` ties the return type to the specific property being accessed. TypeScript infers `T[K]` from the call-site argument, preserving the exact property type rather than widening to a union of all property types.

**Beginner-Friendly Explanation.** `T[K]` means **“the type of whatever property `K` selects from `T`.”** In a function `getProperty(obj, key)`, the return type is `T[K]`, so passing `"name"` gives `string` and passing `"age"` gives `number`.

### Purposes

- **To preserve the precise return type** of property access in generic functions.
- **To constrain keys** so only valid properties can be accessed.
- **To type setters and getters** with matching parameter and return types.
- **To implement type-safe property manipulation** without `any` or assertions.
- **To build reusable accessors** that work across many object types.

### Syntax Rules and Structure

**General syntax:**

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

**Component breakdown:**

- `<T, K extends keyof T>` — `K` is constrained to keys of `T`.
- `obj: T` — the source object.
- `key: K` — the specific key.
- `: T[K]` — the return type matching the selected property.
- `obj[key]` — the runtime access; its inferred type is `T[K]`.

**Setter form:**

```typescript
function setProperty<T, K extends keyof T>(obj: T, key: K, value: T[K]): void {
  obj[key] = value;
}
```

**Syntax rules.**

- `K` must be constrained with `extends keyof T`.
- The return type `T[K]` preserves the property type.
- The setter's `value` parameter is typed `T[K]`.
- Invalid keys are caught at compile time.

**Constraints and limitations.**

- Without the `K extends keyof T` constraint, `obj[key]` is an error.
- In some generic positions, TypeScript may widen `T[K]` to the constraint, losing precision.
- Indexed access on optional properties includes `undefined`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type-safe getter

```typescript
// example_tk_getter.ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "Ada", age: 36 };

const name = getProperty(person, "name"); // string
const age = getProperty(person, "age");   // number

console.log(name.toUpperCase()); // Expected: ADA
console.log(age.toFixed(1));     // Expected: 36.0

// getProperty(person, "email"); // ❌ Error: not a key of person.
```

**Expected Output:**
```
ADA
36.0
```

**Why this output occurs.** `K` is inferred as the literal `"name"` or `"age"`, and `T[K]` resolves to `string` or `number` respectively. String and number methods are available without casts.

---

#### Example 2: Type-safe setter

```typescript
// example_tk_setter.ts
function setProperty<T, K extends keyof T>(obj: T, key: K, value: T[K]): void {
  obj[key] = value;
}

const person = { name: "Ada", age: 36 };

setProperty(person, "name", "Grace"); // ✅ string accepted
console.log(person.name); // Expected: Grace

// setProperty(person, "age", "old"); // ❌ Error: string not assignable to number.
```

**Expected Output:**
```
Grace
```

**Why this output occurs.** The `value` parameter is typed `T[K]`. For `"name"`, it is `string`; for `"age"`, it is `number`. Passing a string where a number is expected is rejected.

---

### Real-World Cases

- **Utility functions**: `pick`, `pluck`, and `get` in libraries.
- **Form libraries**: binding field names to value types.
- **ORM accessors**: typed column getters and setters.

---

## 3. Property Lookup Types

### Definitions

**Core Definition.** **Property lookup types** is another name for indexed access types when they are used to **retrieve the type of a named property**. The term emphasizes the lookup semantics: given a type and a property name, return that property's type.

**Technical Definition.** A property lookup `T["key"]` is an indexed access type whose key is a **string literal**. It may be combined with `keyof` (`T[keyof T]`) to retrieve the union of all property types, or with a union of literals (`T["a" | "b"]`) to retrieve a union of selected property types. Property lookup types are used inside mapped types (`T[P]`) to preserve value types during transformation.

**Beginner-Friendly Explanation.** Property lookup types are **looking up a property's type by name**. `Person["name"]` is a property lookup. `Person[keyof Person]` looks up every property and unions the results.

### Purposes

- **To retrieve a named property's type** without redefining it.
- **To retrieve the union of all property types** via `keyof`.
- **To preserve value types inside mapped types** (`T[P]`).
- **To build `Pick` and `Omit`-style utilities** that carry property types forward.
- **To express relationships between keys and value types** in APIs.

### Syntax Rules and Structure

**Named property lookup:**

```typescript
type NameType = Person["name"]; // string
```

**All-property union:**

```typescript
type AllValues = Person[keyof Person]; // string | number | boolean
```

**Mapped-type lookup:**

```typescript
type Partial<T> = { [P in keyof T]?: T[P] };
```

**Component breakdown:**

- `Person["name"]` — single property lookup.
- `Person[keyof Person]` — union of all property types.
- `T[P]` — in a mapped type, the value type at key `P`.

**Syntax rules.**

- Property names must be string literals, `keyof` results, or unions thereof.
- `T[P]` in mapped types preserves the original property type.
- Property lookup types cannot use dot notation.

**Constraints and limitations.**

- Indexing a non-existent property errors.
- `keyof T` on a type with a broad index signature yields `string | number`, losing specific keys.

### Annotated Complete Step by Step Code Examples

#### Example 1: Named property lookup

```typescript
// example_property_lookup.ts
interface Product {
  id: string;
  price: number;
  inStock: boolean;
}

type PriceType = Product["price"];       // number
type IdOrPrice = Product["id" | "price"]; // string | number

const price: PriceType = 99.99;
console.log(price); // Expected: 99.99
```

**Expected Output:**
```
99.99
```

**Why this output occurs.** `Product["price"]` is `number`; `Product["id" | "price"]` is `string | number`.

---

#### Example 2: Property lookup inside a mapped type

```typescript
// example_property_mapped.ts
type Nullable<T> = { [P in keyof T]: T[P] | null };

interface User {
  name: string;
  age: number;
}

type NullableUser = Nullable<User>;
// { name: string | null; age: number | null }

const user: NullableUser = { name: null, age: 36 };
console.log(user.name); // Expected: null
```

**Expected Output:**
```
null
```

**Why this output occurs.** `T[P]` preserves the original property type (`string` for `name`, `number` for `age`), and `| null` is added to each. This is the canonical use of property lookup inside mapped types.

---

### Real-World Cases

- **`Partial<T>`, `Readonly<T>`, `Pick<T, K>`** — all use `T[P]`.
- **API serializers**: mapping response fields to nullable or optional types.
- **Form models**: deriving value types from a base entity.

---

## 4. Array Element Types

### Definitions

**Core Definition.** **Array element types** are extracted from arrays, tuples, and numeric index signatures using indexed access with `number`: `ArrayType[number]`. This yields the **union of all element types**.

**Technical Definition.** Arrays have a numeric index signature, so `T[number]` retrieves the element type. For tuples, `T[number]` yields the union of all element types; for a specific tuple position, use `T[0]`, `T[1]`, etc. For objects with a numeric index signature, `T[number]` yields the value type of that signature. `Array[number]` is the canonical trick for extracting element types from array literals and `as const` arrays.

**Beginner-Friendly Explanation.** `Array[number]` means **“give me the type of whatever is inside this array.”** For `string[]`, `string[][number]` is `string`. For a tuple `[string, number]`, `T[number]` is `string | number`. For a specific position, `T[0]` is `string`.

### Purposes

- **To extract the element type from an array type** without manual duplication.
- **To derive literal unions from `as const` arrays** for enum-like alternatives.
- **To extract specific tuple element types** by numeric position.
- **To retrieve the value type of a numeric index signature**.
- **To build generic utilities** that operate on arrays and tuples.

### Syntax Rules and Structure

**Element union from an array:**

```typescript
type Element = string[][number]; // string
```

**Element union from a tuple:**

```typescript
type T = [string, number][number]; // string | number
```

**Specific tuple position:**

```typescript
type First = [string, number][0]; // string
```

**From a value with `typeof`:**

```typescript
const roles = ["admin", "editor"] as const;
type Role = (typeof roles)[number]; // "admin" | "editor"
```

**Component breakdown:**

- `T[number]` — all numeric-indexed values.
- `T[0]`, `T[1]` — specific tuple positions.
- `(typeof array)[number]` — element union from an array value.

**Syntax rules.**

- Arrays and tuples have numeric index signatures, so `[number]` is valid.
- Tuples also support specific numeric indices (`[0]`, `[1]`).
- `as const` preserves literal element types for literal unions.

**Constraints and limitations.**

- `T[number]` on a tuple gives the **union** of all element types, not a specific one.
- Without `as const`, array literals widen to `string[]`/`number[]`, losing literal precision.
- `noUncheckedIndexedAccess` adds `| undefined` to `T[number]` reads on arrays.

### Annotated Complete Step by Step Code Examples

#### Example 1: Array element type from `typeof`

```typescript
// example_array_element.ts
const roles = ["admin", "editor", "contributor"] as const;

type Roles = typeof roles;          // readonly ["admin", "editor", "contributor"]
type Role = Roles[number];          // "admin" | "editor" | "contributor"

const role: Role = "admin";
console.log(role); // Expected: admin

// const bad: Role = "owner"; // ❌ Error: not assignable.
```

**Expected Output:**
```
admin
```

**Why this output occurs.** `as const` preserves the literal tuple type. `Roles[number]` extracts the union of element literals, producing `"admin" | "editor" | "contributor"`.

---

#### Example 2: Tuple element types

```typescript
// example_tuple_element.ts
type Pair = [string, number];

type Both = Pair[number];  // string | number
type First = Pair[0];      // string
type Second = Pair[1];     // number

const first: First = "hello";
const second: Second = 42;

console.log(first, second); // Expected: hello 42
```

**Expected Output:**
```
hello 42
```

**Why this output occurs.** `Pair[number]` indexes the tuple with the numeric index signature, producing `string | number`. `Pair[0]` and `Pair[1]` retrieve the specific element types.

---

#### Example 3: Numeric index signature

```typescript
// example_numeric_index.ts
interface NumberMap {
  [key: number]: string;
  name: string; // string keys are separate
}

type NumericValues = NumberMap[number]; // string

const value: NumericValues = "hello";
console.log(value); // Expected: hello
```

**Expected Output:**
```
hello
```

**Why this output occurs.** `NumberMap[number]` retrieves the value type of the numeric index signature, which is `string`. String-keyed properties are not included in the numeric lookup.

---

### Real-World Cases

- **Enum-like literal unions**: `const ROLES = [...] as const; type Role = (typeof ROLES)[number]`.
- **Tuple unpacking**: extracting positional types from fixed-length tuples.
- **Generic array utilities**: `type ElementOf<T extends any[]> = T[number]`.

---

## 5. Nested Indexed Access

### Definitions

**Core Definition.** **Nested indexed access** chains multiple indexed access types to **descend into nested structures**: `T["a"]["b"]` retrieves the type of `b` inside the type of `a`. It is the type-level equivalent of `obj.a.b`.

**Technical Definition.** Nested indexed access applies `[]` repeatedly, each bracket descending one level into the type structure. For arrays and tuples, numeric indices may be chained: `T["items"][0]` retrieves the first element type of the `items` property. Chains may combine string literal keys, `number`, and `keyof` at each level.

**Beginner-Friendly Explanation.** Nested indexed access is **drilling down through layers of a type**. `Response["data"]["user"]["name"]` retrieves the type of `name` inside `user` inside `data` inside `Response`. You can also drill into arrays with `[number]` or `[0]`.

### Purposes

- **To extract deeply nested property types** without redefining the entire structure.
- **To derive types from API response shapes** at any depth.
- **To access tuple element types inside object properties**.
- **To generalize nested lookups** with path-tuple helper types.
- **To keep derived types synchronized** with deeply nested source types.

### Syntax Rules and Structure

**Chained property access:**

```typescript
type DeepType = T["outer"]["inner"]["deepest"];
```

**Array descent:**

```typescript
type FirstItem = T["items"][0];
type Element = T["items"][number];
```

**Mixed chain:**

```typescript
type Name = Response["users"][number]["name"];
```

**Component breakdown:**

- Each `[...]` descends one level.
- `"key"` selects an object property.
- `number` selects the element union of an array.
- `0`, `1`, … select specific tuple positions.

**Syntax rules.**

- Each bracket must be valid for the type at that level.
- Arrays require `number` or a numeric literal; objects require string literal keys.
- Chains may be arbitrarily deep, subject to compiler recursion limits.

**Constraints and limitations.**

- A single bracket only descends one level; deep access requires repeated brackets.
- Optional properties introduce `undefined` at each level.
- Deep chains can produce complex error messages.

### Annotated Complete Step by Step Code Examples

#### Example 1: Nested object lookup

```typescript
// example_nested_object.ts
interface Response {
  data: {
    user: {
      name: string;
      age: number;
    };
  };
}

type NameType = Response["data"]["user"]["name"]; // string
type UserType = Response["data"]["user"];         // { name: string; age: number }

const name: NameType = "Ada";
console.log(name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** Each bracket descends one level: `Response["data"]` is the data object, `["user"]` is the user object, and `["name"]` is `string`.

---

#### Example 2: Nested array lookup

```typescript
// example_nested_array.ts
interface ArtworkSearchResponse {
  artists: {
    name: string;
    artworks: {
      title: string;
      price: number;
    }[];
  }[];
}

type InferredArtwork =
  ArtworkSearchResponse["artists"][number]["artworks"][number];

const artwork: InferredArtwork = { title: "Mona Lisa", price: 1000000 };
console.log(artwork.title); // Expected: Mona Lisa
```

**Expected Output:**
```
Mona Lisa
```

**Why this output occurs.** `["artists"]` selects the array; `[number]` gets the artist element type; `["artworks"]` selects the artworks array; `[number]` gets the artwork element type. The result is the artwork object type.

---

### Real-World Cases

- **API response typing**: `Response["data"]["items"][number]`.
- **GraphQL result types**: chaining through nested query result shapes.
- **Configuration trees**: extracting deeply nested configuration value types.

---

## 6. Indexed Access Best Practices

### Definitions

**Core Definition.** **Indexed access best practices** are guidelines for using `T[K]` and related lookup types **safely, precisely, and performantly**, avoiding unsafe widening, missing-key crashes, and unnecessary type duplication.

**Technical Definition.** Best practices center on **constraining keys with `keyof`**, **preserving precise property types with `T[K]`**, **using `number` for array elements and literals for tuple positions**, **enabling `noUncheckedIndexedAccess` for index-signature safety**, **avoiding non-existent keys and unsafe widening in generics**, and **composing with `keyof`, `typeof`, and mapped types** rather than duplicating types manually.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep indexed access safe and useful. The big rules are: always constrain the key to `keyof T`; use `T[K]` to keep exact property types; use `[number]` for array elements; turn on `noUncheckedIndexedAccess` when working with dictionaries; and avoid looking up keys that might not exist.

### Purposes

- **To preserve precise property types** in generic APIs.
- **To catch invalid keys at compile time** with `K extends keyof T`.
- **To prevent undefined-index crashes** with `noUncheckedIndexedAccess`.
- **To avoid duplicating property types** by deriving them from the source.
- **To keep derived types synchronized** with changes to the source type.

### Syntax Rules and Structure

Best practices are conventions that interact with indexed access syntax. The core rules are:

1. **Constrain keys with `keyof`.** Use `K extends keyof T` to ensure only valid keys are permitted.
2. **Use `T[K]` to preserve the precise property type** in return types and setter parameters.
3. **Use `[number]` for array element types** and specific numeric literals for tuple positions.
4. **Enable `noUncheckedIndexedAccess` for dictionary-like access.** It adds `| undefined` to indexed reads, forcing presence checks.
5. **Avoid non-existent keys.** Indexed access only accepts keys that exist on the source type.
6. **Avoid unnecessary widening in generics.** When `T[K]` widens to the constraint, restructure generics or use the specific property type directly.
7. **Use `as const` for literal array unions.** Without `as const`, array literals widen and lose literal element types.
8. **Prefer derived types over manual duplication.** Use indexed access instead of redefining property types.

**Component breakdown (indexed access decision tree):**

```
What are you trying to extract?
  ├── A named property type → Use T["key"].
  ├── A union of selected properties → Use T["a" | "b"].
  ├── All property types → Use T[keyof T].
  ├── An array element type → Use T[number].
  ├── A tuple element type → Use T[0], T[1], etc.
  └── A deeply nested type → Chain brackets: T["a"]["b"].
```

**Constraints and limitations.**

- `noUncheckedIndexedAccess` can be noisy in hot loops; use non-null assertions only after bounds checks.
- Generic indexed access can widen to the constraint in some positions.
- Deep chains can produce complex error messages.

### Annotated Complete Step by Step Code Examples

#### Example 1: `noUncheckedIndexedAccess` for dictionary safety

```typescript
// example_best_unchecked.ts
// Compiled with: npx tsc --strict --noUncheckedIndexedAccess

interface TagsByPost {
  [postId: string]: string[];
}

function firstTag(tagsByPost: TagsByPost, postId: string): string {
  const tags = tagsByPost[postId]; // string[] | undefined
  if (!tags) return "UNTAGGED";
  return tags[0]?.toUpperCase() ?? "UNTAGGED";
}

console.log(firstTag({ p1: ["typescript"] }, "p1")); // Expected: TYPESCRIPT
console.log(firstTag({}, "missing"));                // Expected: UNTAGGED
```

**Expected Output:**
```
TYPESCRIPT
UNTAGGED
```

**Why this output occurs.** `noUncheckedIndexedAccess` adds `| undefined` to `tagsByPost[postId]` and `tags[0]`. The guard and optional chaining handle missing keys safely, preventing runtime crashes.

---

#### Example 2: `keyof` constraint with `T[K]`

```typescript
// example_best_keyof_tk.ts
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((item) => item[key]);
}

const users = [
  { name: "Ada", age: 36 },
  { name: "Grace", age: 85 },
];

const names = pluck(users, "name"); // string[]
const ages = pluck(users, "age");   // number[]

console.log(names); // Expected: [ 'Ada', 'Grace' ]
console.log(ages);  // Expected: [ 36, 85 ]
```

**Expected Output:**
```
[ 'Ada', 'Grace' ]
[ 36, 85 ]
```

**Why this output occurs.** `K` is constrained to `keyof T`. `T[K][]` preserves the selected property type in the array. The result is `string[]` for `"name"` and `number[]` for `"age"`.

---

#### Example 3: `as const` for literal union extraction

```typescript
// example_best_as_const.ts
const ACTIONS = ["create", "update", "delete"] as const;
type Action = (typeof ACTIONS)[number]; // "create" | "update" | "delete"

function perform(action: Action) {
  console.log(`Performing ${action}`);
}

perform("create"); // Expected: Performing create
// perform("read"); // ❌ Error: not assignable to Action.
```

**Expected Output:**
```
Performing create
```

**Why this output occurs.** `as const` preserves the literal tuple. `(typeof ACTIONS)[number]` extracts the union of literal elements. Without `as const`, the array would widen to `string[]` and `Action` would be `string`.

---

### Real-World Cases

- **Type-safe property utilities**: `pluck`, `pick`, `get`, and `set` functions.
- **Dictionary safety**: enabling `noUncheckedIndexedAccess` for configuration maps and caches.
- **Enum-like literal unions**: deriving unions from `as const` arrays instead of enums.
- **API response modeling**: chaining indexed access to derive nested response types.

---

## References

- Documentation - Indexed Access Types — https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- Documentation - TypeScript 2.1: keyof and Lookup Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html
- Documentation - Creating Types from Types — https://www.typescriptlang.org/docs/handbook/2/types-from-types.html
- Documentation - Keyof Type Operator — https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- Documentation - Typeof Type Operator — https://www.typescriptlang.org/docs/handbook/2/typeof-types.html
- Documentation - Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- Documentation - Utility Types (Pick, Omit, Record) — https://www.typescriptlang.org/docs/handbook/utility-types.html
- Total TypeScript - This Crazy Syntax Lets You Get An Array Element's Type — https://www.totaltypescript.com/get-the-type-of-an-array-element
- TypeScript Wiki - Index Types (kamote learning notes) — https://raw.githubusercontent.com/kamote/learning-notes/2e7499f4340bd1d8e9826eb2e6d64efd8acccc61/javascript/typescript/Index-types.md
- pproenca/dot-skills - Enable noUncheckedIndexedAccess for Index Safety — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/migrate-js-to-modern-typescript/references/strict-no-unchecked-indexed-access.md
- TSConfig Reference - noUncheckedIndexedAccess — https://www.typescriptlang.org/tsconfig/noUncheckedIndexedAccess.html
- Microsoft/TypeScript PR #11929 - Static types for dynamically named properties — https://github.com/microsoft/TypeScript/pull/11929
- Microsoft/TypeScript PR #30769 - Improve soundness of indexed access types — https://github.com/microsoft/TypeScript/pull/30769