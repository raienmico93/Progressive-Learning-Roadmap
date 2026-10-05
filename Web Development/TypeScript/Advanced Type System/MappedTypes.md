# TypeScript Mapped Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Mapped types are a TypeScript type-level feature that creates new object types by iterating over the keys of an existing type and transforming each property according to a defined rule. They are TypeScript's equivalent of JavaScript's `Array.map()`, but applied to type properties instead of array values.

**Technical Definition**
A mapped type is a generic type that uses a union of property keys (frequently created via `keyof`) to iterate through keys and create a new type. The syntax uses an index signature-like structure with the `in` keyword: `{ [P in keyof T]: T[P] }`. Mapped types support property modifiers (`readonly`, `?`) with `+` and `-` prefixes for adding or removing modifiers, and since TypeScript 4.1, key remapping via the `as` clause for filtering and transforming property names. The `exactOptionalPropertyTypes` compiler option (TypeScript 4.4+) further refines mapped type behavior by distinguishing between missing optional properties and explicitly `undefined` optional properties.

**Beginner-Friendly Explanation**
Mapped types are a way to create new types by transforming the properties of an existing type. If you have a `User` type with `name` and `age`, you can use a mapped type to create a `PartialUser` type where all properties are optional, or a `ReadonlyUser` type where all properties are readonly. The syntax `{ [P in keyof T]: T[P] }` says: "for each property `P` in type `T`, create a property with the same name and type `T[P]`." You can then add modifiers or transform the property types. Mapped types are the foundation for TypeScript's built-in utility types like `Partial<T>`, `Readonly<T>`, `Pick<T, K>`, and `Record<K, V>`.

### Key Characteristics

- **Key iteration**: Uses `in keyof T` to iterate over all keys of a type.
- **Property transformation**: Each property's type can be transformed via `T[P]` and conditional types.
- **Modifier control**: `readonly` and `?` modifiers can be added or removed with `+` and `-` prefixes.
- **Key remapping**: Since TypeScript 4.1, the `as` clause enables filtering, renaming, and prefixing/suffixing keys.
- **Homomorphic preservation**: Mapped types of the form `{ [P in keyof T]: ... }` are homomorphic—they preserve the structure and modifiers of the source type.
- **Exact optional properties**: With `exactOptionalPropertyTypes`, mapped types distinguish between missing keys and explicit `undefined` values.
- **Compile-time only**: Mapped types are erased at runtime; they exist only during type checking.

### Prerequisites

- Solid understanding of TypeScript generics (type parameters, constraints)
- Familiarity with `keyof` and indexed access types (`T[K]`)
- Understanding of union types and conditional types
- Familiarity with utility types (`Partial`, `Readonly`, `Pick`, `Record`)

### Related Programming Areas

- **Utility Types**: `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record` are all mapped types.
- **Type-Level Programming**: Mapped types are a core tool for type transformations.
- **Key Remapping**: The `as` clause enables dynamic key transformations and filtering.
- **Conditional Types**: Mapped types often combine with conditional types for property-level transformations.
- **Homomorphic Mappings**: Mapped types that preserve the structure of their source type.

### Core Concepts / Features

1. Iterating Over Keys Using the `in keyof` Syntax
2. Property Transformation and Structural Generation
3. Modifiers for Altering Property States: Adding/Removing `readonly` and Optionality (`+` / `-` Prefixes)
4. Exact Optional Properties Mapping Behavior (Handling Explicit `undefined` vs Missing Keys)
5. Key Remapping Using the `as` Clause to Filter or Transform Property Keys Dynamically


## 1. Iterating Over Keys Using the `in keyof` Syntax

### Definitions

**Core Definition**
The `in keyof` syntax is the foundation of mapped types. It iterates over the union of property names produced by `keyof T`, creating a new type with the same keys but potentially different property types. The syntax `{ [P in keyof T]: NewType }` creates a type where each property `P` of `T` is mapped to `NewType`.

**Technical Definition**
The mapped type syntax `{ [P in K]: X }` iterates over a union of property keys `K` and creates a new object type with those keys. When `K` is `keyof T`, the mapped type is **homomorphic**—it preserves the structure, modifiers, and optionality of the source type `T`. The type parameter `P` represents each key in the union, and `T[P]` (indexed access) refers to the type of that property. The mapped type `{ [P in keyof T]: T[P] }` creates an identical copy of `T`. The `in` keyword is what distinguishes mapped types from index signatures: index signatures use `[key: string]` (a fixed key type), while mapped types use `[P in K]` (an iterated union of keys). Since TypeScript 2.9, the `keyof` operator and mapped types support `number` and `symbol` named properties, not just strings.

**Beginner-Friendly Explanation**
The `in keyof` syntax is the heart of mapped types. Think of it as a "for each" loop, but for types. `{ [P in keyof T]: ... }` says: "for each property `P` in type `T`, create a property with the same name." The `P` is a type variable that takes on each key of `T` one at a time. Inside the mapped type, you can use `T[P]` to refer to the type of the current property. If you write `{ [P in keyof T]: T[P] }`, you get an exact copy of `T`. If you write `{ [P in keyof T]: boolean }`, you get a type where every property is a boolean. This is how TypeScript's utility types like `Partial<T>` and `Readonly<T>` work under the hood.

### Purposes

- To iterate over all keys of a type and create a new type with transformed properties.
- To serve as the foundation for TypeScript's built-in utility types (`Partial`, `Readonly`, `Pick`, `Record`).
- To enable DRY (Don't Repeat Yourself) type definitions that automatically adapt when the source type changes.
- To provide homomorphic type transformations that preserve the structure of the source type.
- To support type-level mapping over `string`, `number`, and `symbol` keys.

### Syntax Rules and Structure

**General Syntax: Basic Mapped Type**

```typescript
type MappedType<T> = {
  [P in keyof T]: T[P];
};
```

**Component Breakdown**
- `[P in keyof T]`: The key iteration clause. `P` takes on each key of `T`.
- `: T[P]`: The property type. `T[P]` is the indexed access type for the current key.
- `MappedType<T>`: A generic type that transforms `T`.

**General Syntax: Modifying Property Types**

```typescript
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};
```

**Component Breakdown**
- Each property type becomes `T[P] | null`.

**General Syntax: Using a Union of Keys**

```typescript
type PartialByKeys<T, K extends keyof T> = {
  [P in K]?: T[P];
};
```

**Component Breakdown**
- `[P in K]`: Iterates over a subset of keys `K` (not all of `T`).
- This creates a non-homomorphic mapped type.

**General Syntax: Mapped Type Over a Union**

```typescript
type Flags = {
  [K in "a" | "b" | "c"]: boolean;
};
// { a: boolean; b: boolean; c: boolean }
```

**Component Breakdown**
- The union of keys can be any union of string, number, or symbol literal types.

**Syntax Rules**

- The `in` keyword introduces the key iteration.
- `keyof T` produces the union of `T`'s property names.
- `P` is a type parameter that represents each key.
- `T[P]` (indexed access) refers to the type of the current property.
- Homomorphic mapped types (those using `keyof T`) preserve modifiers and optionality.
- The key union can include `string`, `number`, and `symbol` keys (TypeScript 2.9+).
- Mapped types can be nested and combined with conditional types.

**Constraints and Limitations**

- The key union must be a union of literal types (not a general `string`).
- Mapped types cannot be used at runtime; they are compile-time only.
- Non-homomorphic mapped types (those using an arbitrary key union) do not preserve modifiers.
- `keyof` on a type with no properties produces `never`, resulting in an empty mapped type.
- Index signatures in the source type affect the `keyof` result (e.g., `string | number` for string index signatures).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Mapped Type

```typescript
// Step 1: Define a source type.
interface User {
  id: number;
  name: string;
  email: string;
}

// Step 2: Create a mapped type that makes all properties boolean.
type Booleanify<T> = {
  [P in keyof T]: boolean;
};

type BooleanUser = Booleanify<User>;
// { id: boolean; name: boolean; email: boolean }

// Step 3: Use the mapped type.
const flags: BooleanUser = {
  id: true,
  name: false,
  email: true,
};

console.log(flags);  // { id: true, name: false, email: true }

// Step 4: Create a mapped type that makes all properties nullable.
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

type NullableUser = Nullable<User>;
// { id: number | null; name: string | null; email: string | null }

const nullableUser: NullableUser = {
  id: 1,
  name: null,
  email: "alice@example.com",
};

console.log(nullableUser);  // { id: 1, name: null, email: 'alice@example.com' }

// Step 5: Homomorphic mapped type (preserves modifiers).
interface PartialUser {
  id: number;
  name?: string;
  readonly email: string;
}

type Copy<T> = { [P in keyof T]: T[P] };
type CopiedUser = Copy<PartialUser>;
// Same as PartialUser — preserves ? and readonly modifiers.

console.log("Mapped types complete.");
```

**Expected Output:**
```
{ id: true, name: false, email: true }
{ id: 1, name: null, email: 'alice@example.com' }
Mapped types complete.
```

**Why This Output Occurs:** The `Booleanify<T>` mapped type iterates over each key of `T` and changes the property type to `boolean`. The `Nullable<T>` mapped type adds `| null` to each property type. The `Copy<T>` homomorphic mapped type preserves the original modifiers (`?` and `readonly`).

#### Example 2: Mapped Types Over Unions and Number Keys

```typescript
// Step 1: Mapped type over a union of literal keys.
type Flags = {
  [K in "read" | "write" | "execute"]: boolean;
};

const permissions: Flags = {
  read: true,
  write: false,
  execute: true,
};

console.log(permissions);  // { read: true, write: false, execute: true }

// Step 2: Mapped type with number keys (TypeScript 2.9+).
type NumberKeys = {
  [K in 0 | 1 | 2]: string;
};

const tupleLike: NumberKeys = {
  0: "zero",
  1: "one",
  2: "two",
};

console.log(tupleLike);  // { '0': 'zero', '1': 'one', '2': 'two' }

// Step 3: Mapped type over an array (produces tuple/array).
type ReadonlyArray<T> = {
  readonly [P in keyof T]: T[P];
};

const arr: ReadonlyArray<number[]> = [1, 2, 3];
// arr.push(4);  // ❌ Error: Property 'push' does not exist on type 'readonly number[]'.

console.log(arr);  // [1, 2, 3]

console.log("Mapped types over unions complete.");
```

**Expected Output:**
```
{ read: true, write: false, execute: true }
{ '0': 'zero', '1': 'one', '2': 'two' }
[ 1, 2, 3 ]
Mapped types over unions complete.
```

**Why This Output Occurs:** The `Flags` mapped type iterates over the union `"read" | "write" | "execute"`. The `NumberKeys` type iterates over numeric literals `0 | 1 | 2`. The `ReadonlyArray<T>` mapped type transforms an array type into a readonly array type, preserving the element type.

### Real-World Cases

**Case 1: Form Libraries**
Form libraries use mapped types to create form value types where all properties are optional or nullable, enabling partial form updates.

**Case 2: API Response Typing**
API clients use mapped types to transform response types, making all properties readonly or adding error types to each field.

**Case 3: State Management**
State management libraries use mapped types to create action types from state shapes, ensuring type safety across reducers.

**Case 4: Configuration Validation**
Configuration validators use mapped types to create validation rule types for each configuration key.

**Case 5: Database Schema Types**
ORMs use mapped types to transform entity types into query result types, adding metadata or transforming field types.

**Case 6: Testing Utilities**
Test utilities use mapped types to create mock types where all properties are jest mock functions.

### References

- TypeScript Handbook: Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript 2.9 Release Notes: Support number and symbol named properties with keyof and mapped types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-9.html
- TypeScript Playground: Mapped Types — https://www.typescriptlang.org/play/typescript/meta-types/mapped-types.ts.html
- TypeScript 2.1 Release Notes: Mapped Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html


## 2. Property Transformation and Structural Generation

### Definitions

**Core Definition**
Property transformation is the process of changing the type of each property in a mapped type. The indexed access type `T[P]` provides the original property type, which can be transformed using union types, conditional types, or other type operators. Structural generation refers to the creation of entirely new type structures from an existing type's keys and values.

**Technical Definition**
Inside a mapped type, the property type expression `T[P]` can be transformed in several ways: (1) wrapping in another type (`T[P] | null`, `Promise<T[P]>`), (2) applying a conditional type (`T[P] extends Function ? never : T[P]`), (3) using the property type in a more complex expression (`{ value: T[P] }`), or (4) replacing it entirely with a fixed type (`boolean`, `string`). The indexed access `T[P]` is the bridge between the source and target types. Structural generation uses the keys of the source type to create a new structure—for example, generating getter/setter methods or transforming a data type into an API client type. The transformation can depend on the key (`P`), the property type (`T[P]`), or both.

**Beginner-Friendly Explanation**
Property transformation is what happens inside a mapped type. You take the original property type (`T[P]`) and change it. You can add `| null` to make it nullable, wrap it in a `Promise` or `Array`, or use a conditional type to change it based on what it is. For example, you might transform all function properties into `never` (removing them) or all properties into getter methods. Structural generation goes further: you can create entirely new types from an existing type's keys. For example, from a type with `name` and `age`, you can generate a type with `getName` and `getAge` methods. This is how libraries like React Hook Form generate form field types from form value types.

### Purposes

- To transform property types (e.g., making them nullable, readonly, or wrapped in another type).
- To generate new type structures (e.g., getters, setters, API clients) from existing type keys.
- To apply conditional logic to individual properties based on their types.
- To create domain-specific type transformations (e.g., form field types, query result types).
- To reduce type duplication by deriving types from a single source of truth.

### Syntax Rules and Structure

**General Syntax: Type Wrapping**

```typescript
type Wrapped<T> = {
  [P in keyof T]: { value: T[P] };
};
```

**Component Breakdown**
- Each property becomes an object with a `value` property.

**General Syntax: Conditional Property Transformation**

```typescript
type FunctionKeysToNever<T> = {
  [P in keyof T]: T[P] extends Function ? never : T[P];
};
```

**Component Breakdown**
- Function properties become `never` (effectively removing them).

**General Syntax: Property Type Replacement**

```typescript
type AllStrings<T> = {
  [P in keyof T]: string;
};
```

**Component Breakdown**
- All property types become `string`.

**General Syntax: Structural Generation (Getters)**

```typescript
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};
```

**Component Breakdown**
- Generates getter methods for each property using key remapping.

**Syntax Rules**

- `T[P]` accesses the original property type.
- The property type can be transformed with unions, intersections, conditional types, or utility types.
- The transformation can depend on `P` (the key) or `T[P]` (the property type).
- Conditional types can filter or transform properties based on their types.
- Key remapping (`as`) can be combined with property transformation.
- The transformed type can be a completely different structure.

**Constraints and Limitations**

- The transformation must be expressible at the type level (no runtime logic).
- Complex conditional types can produce unreadable error messages.
- The transformation cannot depend on runtime values.
- Recursive mapped types can cause compiler performance issues.
- Deeply nested transformations can hit TypeScript's recursion limits.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Property Type Transformations

```typescript
// Step 1: Define a source type.
interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
  getDetails: () => string;
}

// Step 2: Transform all properties to nullable.
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

type NullableProduct = Nullable<Product>;
// All properties are now `T | null`.

// Step 3: Transform all properties to promises.
type Promisify<T> = {
  [P in keyof T]: Promise<T[P]>;
};

type AsyncProduct = Promisify<Product>;
// All properties are now Promise<T>.

// Step 4: Conditional transformation (remove functions).
type RemoveFunctions<T> = {
  [P in keyof T]: T[P] extends Function ? never : T[P];
};

type ProductData = RemoveFunctions<Product>;
// { id: number; name: string; price: number; inStock: boolean; getDetails: never }

// Step 5: Structural generation (getters).
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

type ProductGetters = Getters<Product>;
// { getId: () => number; getName: () => string; getPrice: () => number; ... }

// Step 6: Use the transformed types.
const nullableProduct: NullableProduct = {
  id: 1,
  name: null,
  price: 999,
  inStock: null,
  getDetails: null,
};
console.log(nullableProduct.name);  // null

const productData: ProductData = {
  id: 1,
  name: "Laptop",
  price: 999,
  inStock: true,
  getDetails: undefined as never,
};
console.log(productData.name);  // "Laptop"

console.log("Property transformations complete.");
```

**Expected Output:**
```
null
Laptop
Property transformations complete.
```

**Why This Output Occurs:** The `Nullable<T>` mapped type wraps each property type in `T[P] | null`. The `Promisify<T>` wraps each in `Promise<T[P]>`. The `RemoveFunctions<T>` conditional transformation replaces function types with `never`. The `Getters<T>` mapped type generates getter methods using key remapping.

#### Example 2: Domain-Specific Transformations

```typescript
// Step 1: Define a source type.
interface ApiEndpoints {
  getUser: (id: number) => Promise<User>;
  createUser: (data: UserData) => Promise<User>;
  deleteUser: (id: number) => Promise<void>;
}

interface User {
  id: number;
  name: string;
}

interface UserData {
  name: string;
}

// Step 2: Transform to extract return types.
type ReturnTypes<T> = {
  [P in keyof T]: T[P] extends (...args: any[]) => infer R ? R : never;
};

type EndpointResults = ReturnTypes<ApiEndpoints>;
// { getUser: Promise<User>; createUser: Promise<User>; deleteUser: Promise<void> }

// Step 3: Transform to extract parameter types.
type ParameterTypes<T> = {
  [P in keyof T]: T[P] extends (...args: infer P) => any ? P : never;
};

type EndpointParams = ParameterTypes<ApiEndpoints>;
// { getUser: [id: number]; createUser: [data: UserData]; deleteUser: [id: number] }

// Step 4: Transform to create mock functions.
type MockApi<T> = {
  [P in keyof T]: T[P] extends (...args: any[]) => any
    ? (...args: Parameters<T[P]>) => ReturnType<T[P]>
    : never;
};

// Step 5: Use the transformed types.
const results: EndpointResults = {
  getUser: Promise.resolve({ id: 1, name: "Alice" }),
  createUser: Promise.resolve({ id: 2, name: "Bob" }),
  deleteUser: Promise.resolve(),
};

console.log(await results.getUser);  // { id: 1, name: 'Alice' }

console.log("Domain-specific transformations complete.");
```

**Expected Output:**
```
{ id: 1, name: 'Alice' }
Domain-specific transformations complete.
```

**Why This Output Occurs:** The `ReturnTypes<T>` mapped type uses conditional types with `infer` to extract the return type of each function property. The `ParameterTypes<T>` extracts parameter tuples. These transformations derive new types from the API endpoint structure, reducing duplication.

### Real-World Cases

**Case 1: API Client Generation**
API clients use mapped types to transform endpoint definitions into typed client methods, extracting parameter and return types.

**Case 2: Form Libraries**
Form libraries use mapped types to transform value types into field types, adding validation metadata or transforming types for form inputs.

**Case 3: State Management**
State management libraries use mapped types to generate action creators, selectors, and reducers from a state shape.

**Case 4: ORM Query Builders**
ORMs use mapped types to transform entity types into query result types, adding database-specific metadata.

**Case 5: Testing Utilities**
Test utilities use mapped types to create mock types where all methods are jest mock functions with the correct signatures.

**Case 6: GraphQL Code Generation**
GraphQL code generators use mapped types to transform query result types into TypeScript types, extracting nested data structures.

### References

- TypeScript Handbook: Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript Handbook: Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Handbook: Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- Total TypeScript: Mapped Types — https://www.totaltypescript.com/workshops/type-transformations/mapped-types


## 3. Modifiers for Altering Property States: Adding/Removing `readonly` and Optionality (`+` / `-` Prefixes)

### Definitions

**Core Definition**
Mapped types support two property modifiers: `readonly` (for immutability) and `?` (for optionality). These modifiers can be prefixed with `+` to add them or `-` to remove them. When no prefix is used, `+` is assumed. This enables precise control over property states in transformed types.

**Technical Definition**
In a mapped type, `readonly` and `?` modifiers can be applied to each property. The `+` prefix (or no prefix) adds the modifier, while the `-` prefix removes it. For homomorphic mapped types (`{ [P in keyof T]: ... }`), the modifiers of the source type are preserved by default. The `-?` modifier removes optionality and, under `strictNullChecks`, also removes `undefined` from the property type. The `-readonly` modifier removes readonly, making properties mutable. TypeScript 2.8 introduced the `+` and `-` prefixes for modifiers, and `Required<T>` and `Mutable<T>` are built-in utility types that use these modifiers.

**Beginner-Friendly Explanation**
Mapped types can add or remove `readonly` and `?` modifiers from properties. If you write `{ readonly [P in keyof T]: T[P] }`, you make all properties readonly. If you write `{ -readonly [P in keyof T]: T[P] }`, you remove readonly from all properties. Similarly, `{ [P in keyof T]?: T[P] }` makes all properties optional, and `{ [P in keyof T]-?: T[P] }` makes all properties required. The `+` prefix is optional—if you don't write anything, `+` is assumed. The `-?` modifier is especially important: it not only removes optionality but also removes `undefined` from the property type under `strictNullChecks`, which is how `Required<T>` works.

### Purposes

- To make all properties readonly (immutable) or remove readonly (mutable).
- To make all properties optional or required.
- To create utility types like `Required<T>`, `Readonly<T>`, and `Mutable<T>`.
- To strip modifiers from a type for specific transformations.
- To combine modifier changes with property transformations for complex type manipulations.

### Syntax Rules and Structure

**General Syntax: Adding Modifiers**

```typescript
type ReadonlyPartial<T> = {
  readonly [P in keyof T]?: T[P];
};
```

**Component Breakdown**
- `readonly`: Adds readonly to all properties.
- `?`: Adds optionality to all properties.

**General Syntax: Removing Modifiers**

```typescript
type MutableRequired<T> = {
  -readonly [P in keyof T]-?: T[P];
};
```

**Component Breakdown**
- `-readonly`: Removes readonly.
- `-?`: Removes optionality (and `undefined` under `strictNullChecks`).

**General Syntax: Explicit `+` Prefix**

```typescript
type ExplicitReadonly<T> = {
  +readonly [P in keyof T]: T[P];
};
```

**Component Breakdown**
- `+readonly`: Explicitly adds readonly (same as `readonly` without prefix).

**General Syntax: Combining with Key Remapping**

```typescript
type ReadonlyGetters<T> = {
  +readonly [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};
```

**Component Breakdown**
- Modifiers can be combined with key remapping.

**Syntax Rules**

- `readonly` and `?` are the two modifiers.
- `+` (or no prefix) adds the modifier.
- `-` removes the modifier.
- `-?` removes optionality and `undefined` under `strictNullChecks`.
- `-readonly` removes readonly.
- Homomorphic mapped types preserve source modifiers by default.
- Modifiers can be combined with key remapping (`as`).
- Modifiers apply to every property in the mapped type.

**Constraints and Limitations**

- Modifiers cannot be applied to non-property types (e.g., primitives).
- `-?` and `-readonly` are not available in TypeScript versions before 2.8.
- Removing `?` from a property also removes `undefined` from its type under `strictNullChecks`.
- Modifiers interact with `exactOptionalPropertyTypes` for optional property handling.
- Overusing modifier changes can make types harder to understand.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Adding and Removing Modifiers

```typescript
// Step 1: Define a source type.
interface User {
  readonly id: number;
  name: string;
  email?: string;
}

// Step 2: Add modifiers — make all properties readonly and optional.
type ReadonlyPartial<T> = {
  +readonly [P in keyof T]+?: T[P];
};

type ReadonlyPartialUser = ReadonlyPartial<User>;
// { readonly id?: number; readonly name?: string; readonly email?: string }

// Step 3: Remove modifiers — make all properties mutable and required.
type MutableRequired<T> = {
  -readonly [P in keyof T]-?: T[P];
};

type MutableRequiredUser = MutableRequired<User>;
// { id: number; name: string; email: string }

// Step 4: Use the transformed types.
const readonlyUser: ReadonlyPartialUser = { id: 1, name: "Alice" };
// readonlyUser.name = "Bob";  // ❌ Error: Cannot assign to 'name' because it is a read-only property.

const requiredUser: MutableRequiredUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
};
requiredUser.name = "Bob";  // ✅ Allowed (mutable)

console.log(requiredUser);  // { id: 1, name: 'Bob', email: 'alice@example.com' }

// Step 5: Built-in utility types use these modifiers.
type RequiredUser = Required<User>;
// { id: number; name: string; email: string }

type ReadonlyUser = Readonly<User>;
// { readonly id: number; readonly name: string; readonly email?: string }

console.log("Modifier transformations complete.");
```

**Expected Output:**
```
{ id: 1, name: 'Bob', email: 'alice@example.com' }
Modifier transformations complete.
```

**Why This Output Occurs:** The `ReadonlyPartial<T>` mapped type adds `+readonly` and `+?` to all properties. The `MutableRequired<T>` removes `-readonly` and `-?`. The `Required<User>` utility removes optionality, and `Readonly<User>` adds readonly. The `-?` modifier also removes `undefined` from the property type under `strictNullChecks`.

#### Example 2: Modifier Interaction with Property Types

```typescript
// Step 1: Define a source type with optional and readonly properties.
interface Config {
  readonly apiUrl: string;
  timeout?: number;
  retries?: number;
}

// Step 2: Make everything required and mutable.
type MutableRequired<T> = {
  -readonly [P in keyof T]-?: T[P];
};

type ResolvedConfig = MutableRequired<Config>;
// { apiUrl: string; timeout: number; retries: number }

// Step 3: Make everything optional and readonly.
type OptionalReadonly<T> = {
  +readonly [P in keyof T]+?: T[P];
};

type PartialReadonlyConfig = OptionalReadonly<Config>;
// { readonly apiUrl?: string; readonly timeout?: number; readonly retries?: number }

// Step 4: Remove only optionality (keep readonly).
type RequiredConfig = {
  [P in keyof Config]-?: Config[P];
};

type RequiredOnlyConfig = RequiredConfig;
// { readonly apiUrl: string; timeout: number; retries: number }

// Step 5: Remove only readonly (keep optionality).
type MutableConfig = {
  -readonly [P in keyof Config]: Config[P];
};

type MutableOnlyConfig = MutableConfig;
// { apiUrl: string; timeout?: number; retries?: number }

// Step 6: Use the types.
const resolved: ResolvedConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
};

console.log(resolved);  // { apiUrl: 'https://api.example.com', timeout: 5000, retries: 3 }

console.log("Modifier interaction complete.");
```

**Expected Output:**
```
{ apiUrl: 'https://api.example.com', timeout: 5000, retries: 3 }
Modifier interaction complete.
```

**Why This Output Occurs:** The `MutableRequired<T>` removes both `readonly` and `?`, making all properties mutable and required. The `OptionalReadonly<T>` adds both modifiers. The `RequiredConfig` removes only `?` (keeping `readonly`), and `MutableConfig` removes only `readonly` (keeping `?`). This demonstrates the granular control over modifiers.

### Real-World Cases

**Case 1: Configuration Resolution**
Configuration libraries use `Required<T>` to ensure all configuration values are resolved before use, removing optionality after merging defaults.

**Case 2: Immutable State**
Redux and other state management libraries use `Readonly<T>` to enforce immutability of state objects.

**Case 3: Form Submission**
Form libraries use `Required<T>` to ensure all form fields are present before submission, removing optionality from validated forms.

**Case 4: API Request Bodies**
API clients use `MutableRequired<T>` to ensure request bodies have all required fields and are mutable for serialization.

**Case 5: Partial Updates**
Update functions use `Partial<T>` to accept partial objects, allowing callers to specify only the fields they want to update.

**Case 6: Draft Types**
Immer and similar libraries use `Draft<T>` (which uses modifier removal) to allow mutable operations on immutable state during draft phases.

### References

- TypeScript 2.8 Release Notes: Improved control over mapped type modifiers — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html
- TypeScript Handbook: Mapped Types (Mapping Modifiers) — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers
- TypeScript Handbook: Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- GitHub: TypeScript PR #21935 — https://github.com/microsoft/TypeScript/pull/21935


## 4. Exact Optional Properties Mapping Behavior (Handling Explicit `undefined` vs Missing Keys)

### Definitions

**Core Definition**
With the `exactOptionalPropertyTypes` compiler option (TypeScript 4.4+) enabled, optional properties in mapped types distinguish between "missing" and "explicitly `undefined`." Without this option, `{ name?: string }` is equivalent to `{ name: string | undefined }`, meaning both missing keys and explicit `undefined` are allowed. With `exactOptionalPropertyTypes`, optional properties cannot be assigned `undefined` explicitly—they must either be present with a value or absent entirely.

**Technical Definition**
When `exactOptionalPropertyTypes` is `false` (default), the type `{ name?: string }` is treated as `{ name: string | undefined }`, and both `{}` and `{ name: undefined }` are valid. When `exactOptionalPropertyTypes` is `true`, the type `{ name?: string }` means the property may be missing, but if present, it must be a `string` (not `undefined`). This affects homomorphic mapped types: `{ [P in keyof T]: T[P] }` may preserve the optionality of `T[P]` without adding `| undefined` when `exactOptionalPropertyTypes` is on. However, indexing an optional property within a mapped type may produce `T[P] | undefined` in some cases depending on whether the mapped type is homomorphic. The `-?` modifier removes optionality and, under `strictNullChecks`, also removes `undefined` from the property type. TypeScript issues #60233 and #60138 document assignability issues with homomorphic mapped types and `exactOptionalPropertyTypes`.

**Beginner-Friendly Explanation**
Normally in TypeScript, an optional property like `email?: string` means "the property might be missing, or it might be a string, or it might be `undefined`." With `exactOptionalPropertyTypes` enabled, it means "the property might be missing, but if it's there, it must be a string—not `undefined`." This is a stricter mode that prevents the subtle bug where you pass `{ email: undefined }` instead of just leaving out the property. In mapped types, this distinction can affect how property types are preserved. When you use a homomorphic mapped type like `{ [P in keyof T]: T[P] }`, the optionality is preserved, but the exact treatment of `undefined` depends on whether you're reading the property type directly or wrapping it in another type.

### Purposes

- To distinguish between missing optional properties and explicitly `undefined` properties.
- To enforce stricter type safety around optional properties in mapped types.
- To prevent accidental assignment of `undefined` to optional properties.
- To understand how homomorphic mapped types interact with `exactOptionalPropertyTypes`.
- To handle assignability issues that arise when mapping types with `exactOptionalPropertyTypes`.

### Syntax Rules and Structure

**General Syntax: Optional Property Without `exactOptionalPropertyTypes`**

```typescript
// exactOptionalPropertyTypes: false (default)
interface User {
  name: string;
  email?: string;  // Equivalent to email?: string | undefined
}

const user1: User = { name: "Alice" };                     // ✅
const user2: User = { name: "Bob", email: undefined };    // ✅ (allowed!)
```

**Component Breakdown**
- Optional properties include `undefined` in their type.
- Both missing and explicit `undefined` are allowed.

**General Syntax: Optional Property With `exactOptionalPropertyTypes`**

```typescript
// exactOptionalPropertyTypes: true
interface User {
  name: string;
  email?: string;  // Cannot be explicitly undefined
}

const user1: User = { name: "Alice" };                     // ✅
// const user2: User = { name: "Bob", email: undefined };  // ❌ Error!
// const user3: User = { name: "Bob", email: "bob@example.com" };  // ✅
```

**Component Breakdown**
- Optional properties cannot be explicitly `undefined`.
- Only missing or a defined value is allowed.

**General Syntax: Mapped Type Behavior**

```typescript
// With exactOptionalPropertyTypes: true
type Foo<Type> = { [P in keyof Type]: Type[P] };
type T = Foo<{ a?: 1 }>;
// T = { a?: 1 }  (preserves optionality without adding undefined)
```

**Component Breakdown**
- Homomorphic mapped types preserve the exact optional property behavior.

**General Syntax: Wrapping to Get `undefined`**

```typescript
// With exactOptionalPropertyTypes: true
type Foo<Type> = { [P in keyof Type]: [Type[P]] };
type T = Foo<{ a?: 1 }>;
// T = { a?: [1 | undefined] }  (wrapping adds undefined)
```

**Component Breakdown**
- Wrapping the property type in a tuple forces `undefined` to be included.

**Syntax Rules**

- `exactOptionalPropertyTypes` is a `tsconfig` compiler option (TypeScript 4.4+).
- When `true`, optional properties cannot be explicitly `undefined`.
- Homomorphic mapped types (`{ [P in keyof T]: T[P] }`) preserve optionality.
- Indexing an optional property within a mapped type may or may not include `undefined` depending on context.
- Wrapping the property type in a tuple (`[T[P]]`) forces `undefined` to be included.
- The `-?` modifier removes optionality and `undefined` under `strictNullChecks`.

**Constraints and Limitations**

- `exactOptionalPropertyTypes` requires TypeScript 4.4+.
- Homomorphic mapped types can break assignability with `exactOptionalPropertyTypes` (GitHub issues #60233, #60138).
- Not all libraries and frameworks are compatible with `exactOptionalPropertyTypes`.
- The behavior of optional properties in mapped types can be non-deterministic in some cases (GitHub issue #60717).
- The option can be enabled independently of `strict` mode.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing With and Without `exactOptionalPropertyTypes`

```typescript
// Step 1: Without exactOptionalPropertyTypes.
// tsconfig.json: { "compilerOptions": { "strict": true } }
interface User1 {
  name: string;
  email?: string;
}

const user1a: User1 = { name: "Alice" };
const user1b: User1 = { name: "Bob", email: undefined };  // ✅ Allowed
console.log(user1b.email);  // undefined

// Step 2: With exactOptionalPropertyTypes.
// tsconfig.json: { "compilerOptions": { "strict": true, "exactOptionalPropertyTypes": true } }
interface User2 {
  name: string;
  email?: string;
}

const user2a: User2 = { name: "Alice" };
// const user2b: User2 = { name: "Bob", email: undefined };
// ❌ Error: Type '{ name: string; email: undefined; }' is not assignable to type 'User2'.
//   Property 'email' is optional and cannot be assigned 'undefined'.

// Step 3: Mapped type behavior with exactOptionalPropertyTypes.
type Copy<T> = { [P in keyof T]: T[P] };
type CopiedUser = Copy<User2>;
// CopiedUser = { name: string; email?: string }

const copied: CopiedUser = { name: "Charlie" };
console.log(copied.name);  // "Charlie"

// Step 4: Wrapping the property type to include undefined.
type Wrapped<T> = { [P in keyof T]: [T[P]] };
type WrappedUser = Wrapped<User2>;
// WrappedUser = { name: [string]; email?: [string | undefined] }

const wrapped: WrappedUser = { name: ["Dave"] };
console.log(wrapped.name);  // ["Dave"]

console.log("Exact optional property types demonstrated.");
```

**Expected Output:**
```
undefined
Charlie
[ 'Dave' ]
Exact optional property types demonstrated.
```

**Why This Output Occurs:** Without `exactOptionalPropertyTypes`, `{ email: undefined }` is allowed. With it, `undefined` is rejected for optional properties. The homomorphic `Copy<T>` preserves the optional property behavior. Wrapping `T[P]` in a tuple forces `undefined` to be included in the wrapped type.

#### Example 2: Assignability Issues with Homomorphic Mapped Types

```typescript
// Step 1: Define a type with optional properties.
interface Settings {
  theme: string;
  fontSize?: number;
}

// Step 2: Create a homomorphic mapped type.
type PartialSettings<T> = { [P in keyof T]?: T[P] };

type PartialConfig = PartialSettings<Settings>;
// With exactOptionalPropertyTypes: { theme?: string; fontSize?: number }

// Step 3: Create an instance.
const config: PartialConfig = { theme: "dark" };

// Step 4: Assignability can break with exactOptionalPropertyTypes.
// The following may error with exactOptionalPropertyTypes enabled:
// const settings: Settings = config;
// ❌ Error: Property 'theme' is optional in type 'PartialConfig' but required in type 'Settings'.

// Step 5: Workaround — use Required or explicit undefined.
type ExactPartial<T> = { [P in keyof T]?: T[P] | undefined };
const exactConfig: ExactPartial<Settings> = { theme: "dark" };
const settings: Settings = { theme: "dark", ...exactConfig };

console.log(settings);  // { theme: 'dark' }

console.log("Assignability workaround complete.");
```

**Expected Output:**
```
{ theme: 'dark' }
Assignability workaround complete.
```

**Why This Output Occurs:** With `exactOptionalPropertyTypes`, a mapped type that makes properties optional may not be assignable to the original required-property type. Adding `| undefined` explicitly or using `Required<T>` resolves the assignability issue.

### Real-World Cases

**Case 1: Configuration Merging**
Configuration libraries that merge partial configs with defaults use `exactOptionalPropertyTypes` to distinguish between "not set" and "set to undefined."

**Case 2: Form Data**
Form libraries use `exactOptionalPropertyTypes` to distinguish between an empty field and a field that hasn't been touched.

**Case 3: API Payloads**
API clients use `exactOptionalPropertyTypes` to ensure optional fields are either present with values or absent entirely, not explicitly `undefined`.

**Case 4: State Management**
State management libraries use `exactOptionalPropertyTypes` to enforce strict optional state updates.

**Case 5: Database Updates**
Database update functions use `exactOptionalPropertyTypes` to distinguish between "don't update this field" and "set this field to null/undefined."

### References

- TypeScript 4.4 Release Notes: exactOptionalPropertyTypes — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html#exact-optional-property-types---exactoptionalpropertytypes
- TypeScript `exactOptionalPropertyTypes` Documentation — https://www.typescriptlang.org/tsconfig#exactOptionalPropertyTypes
- GitHub Issue #60717: How does indexing an optional property within a mapped type behave? — https://github.com/microsoft/TypeScript/issues/60717
- GitHub Issue #60233: Certain homomorphic mappings break assignability with exactOptionalPropertyTypes — https://github.com/microsoft/TypeScript/issues/60233
- GitHub Issue #60138: `exactOptionalPropertyTypes` faults the use of `Omit` — https://github.com/microsoft/TypeScript/issues/60138
- Exploring TypeScript: exactOptionalPropertyTypes — https://exploringjs.com/ts/book/ch_optional-props.html


## 5. Key Remapping Using the `as` Clause to Filter or Transform Property Keys Dynamically

### Definitions

**Core Definition**
Key remapping via the `as` clause is a TypeScript 4.1 feature that allows mapped types to transform, rename, or filter property keys during iteration. The syntax `{ [P in keyof T as NewKey]: ... }` uses the `as` clause to compute a new key for each property, enabling three operations in a single pass: filtering keys (by mapping to `never`), renaming keys (by mapping to a different literal), and prefixing/suffixing keys (via template literal types).

**Technical Definition**
The `as` clause appears after the key iteration clause: `[P in keyof T as NewKeyExpression]`. The `NewKeyExpression` can be any type expression that evaluates to a string, number, or symbol literal (or a union of these). If the expression evaluates to `never`, the property is omitted from the result. If it evaluates to a different literal, the property is renamed. Template literal types enable prefixing and suffixing: `` `get${Capitalize<string & P>}` ``. Key remapping combined with conditional types allows filtering based on the property type (`T[P] extends Function ? K : never`). The `as` clause is the most under-used advanced mapped-type feature in real codebases.

**Beginner-Friendly Explanation**
Key remapping lets you change the names of properties while mapping. Instead of just transforming values, you can also transform keys. For example, you can rename all properties to `get` + `Capitalize` of the original name, so `name` becomes `getName`. Or you can filter out properties by mapping them to `never`, which removes them entirely. You can even combine filtering and renaming: for example, keep only non-function properties and rename them to `set` + name. This is incredibly powerful for generating API clients, form field types, and utility types. The `as` clause is what makes mapped types truly dynamic—it lets you create entirely new type structures from existing ones.

### Purposes

- To filter out properties by mapping their keys to `never`.
- To rename properties dynamically using template literal types.
- To prefix or suffix property names (e.g., `get`/`set` prefixes).
- To combine filtering and renaming in a single mapped type.
- To generate event handler names, selector hooks, and API method names from existing types.

### Syntax Rules and Structure

**General Syntax: Basic Key Remapping**

```typescript
type Remapped<T> = {
  [P in keyof T as NewKeyExpression]: T[P];
};
```

**Component Breakdown**
- `as NewKeyExpression`: Computes the new key for each property.
- If the expression is `never`, the property is removed.

**General Syntax: Filtering Keys**

```typescript
type Methods<T> = {
  [K in keyof T as T[K] extends Function ? K : never]: T[K];
};
```

**Component Breakdown**
- Only function properties are kept; others are mapped to `never` (removed).

**General Syntax: Renaming Keys with Template Literals**

```typescript
type Getters<T> = {
  [K in keyof T & string as `get${Capitalize<K>}`]: () => T[K];
};
```

**Component Breakdown**
- `get${Capitalize<K>}`: Prefixes each key with `get` and capitalizes.

**General Syntax: Combining Filter and Rename**

```typescript
type Setters<T> = {
  [K in keyof T & string as T[K] extends Function ? never : `set${Capitalize<K>}`]: (value: T[K]) => void;
};
```

**Component Breakdown**
- Function properties are filtered out; others are renamed with `set` prefix.

**Syntax Rules**

- The `as` clause appears after the `in` clause: `[P in K as NewKey]`.
- The new key expression must evaluate to a string, number, or symbol literal.
- `never` removes the property from the result.
- Template literal types enable prefixing/suffixing.
- Conditional types enable filtering based on property types.
- The `& string` intersection widens the key type for template literal operations.
- Key remapping works with both homomorphic and non-homomorphic mapped types.

**Constraints and Limitations**

- Template literals only work on string keys; `number` and `symbol` keys are dropped.
- The `as` clause cannot depend on runtime values.
- Complex conditional expressions in the `as` clause can produce unreadable errors.
- Key remapping is available in TypeScript 4.1 and later.
- Filtering via `never` removes properties but does not prevent other transformations.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Key Remapping

```typescript
// Step 1: Define a source type.
interface User {
  id: string;
  name: string;
  save(): Promise<void>;
  delete(): Promise<void>;
}

// Step 2: Filter to only methods.
type Methods<T> = {
  [K in keyof T as T[K] extends Function ? K : never]: T[K];
};

type UserMethods = Methods<User>;
// { save: () => Promise<void>; delete: () => Promise<void> }

// Step 3: Rename to getters.
type Getters<T> = {
  [K in keyof T & string as `get${Capitalize<K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// { getId: () => string; getName: () => string; getSave: () => ...; getDelete: () => ... }

// Step 4: Combine filter and rename — only data properties as setters.
type Setters<T> = {
  [K in keyof T & string as T[K] extends Function ? never : `set${Capitalize<K>}`]: (value: T[K]) => void;
};

type UserSetters = Setters<User>;
// { setId: (value: string) => void; setName: (value: string) => void }

// Step 5: Use the remapped types.
const getters: UserGetters = {
  getId: () => "1",
  getName: () => "Alice",
  getSave: () => Promise.resolve(),
  getDelete: () => Promise.resolve(),
};

console.log(getters.getId());  // "1"
console.log(getters.getName()); // "Alice"

console.log("Key remapping complete.");
```

**Expected Output:**
```
1
Alice
Key remapping complete.
```

**Why This Output Occurs:** The `Methods<T>` mapped type filters out non-function properties by mapping their keys to `never`. The `Getters<T>` type renames all properties with a `get` prefix. The `Setters<T>` type filters out function properties and renames the rest with a `set` prefix.

#### Example 2: Dynamic Event Handler Generation

```typescript
// Step 1: Define an event map.
interface EventMap {
  loggedIn: { userId: string };
  loggedOut: { userId: string; timestamp: Date };
  error: { message: string; code: number };
}

// Step 2: Generate event handler prop names.
type EventHandlers<T> = {
  [K in keyof T & string as `on${Capitalize<K>}`]: (payload: T[K]) => void;
};

type AppEventHandlers = EventHandlers<EventMap>;
// {
//   onLoggedIn: (payload: { userId: string }) => void;
//   onLoggedOut: (payload: { userId: string; timestamp: Date }) => void;
//   onError: (payload: { message: string; code: number }) => void;
// }

// Step 3: Use the generated handlers.
const handlers: AppEventHandlers = {
  onLoggedIn: (payload) => console.log(`User ${payload.userId} logged in`),
  onLoggedOut: (payload) => console.log(`User ${payload.userId} logged out`),
  onError: (payload) => console.log(`Error ${payload.code}: ${payload.message}`),
};

handlers.onLoggedIn({ userId: "user-123" });  // "User user-123 logged in"
handlers.onError({ message: "Not found", code: 404 });  // "Error 404: Not found"

// Step 4: Generate selector hooks from a state shape.
interface AppState {
  user: { name: string; age: number };
  theme: "light" | "dark";
}

type SelectorHooks<T> = {
  [K in keyof T & string as `use${Capitalize<K>}`]: () => T[K];
};

type AppSelectors = SelectorHooks<AppState>;
// { useUser: () => { name: string; age: number }; useTheme: () => "light" | "dark" }

console.log("Event handler and selector generation complete.");
```

**Expected Output:**
```
User user-123 logged in
Error 404: Not found
Event handler and selector generation complete.
```

**Why This Output Occurs:** The `EventHandlers<T>` mapped type uses the `as` clause with template literals to generate `on` + `Capitalize` handler names from event names. The `SelectorHooks<T>` type generates `use` + `Capitalize` selector names from state keys. Both demonstrate how key remapping enables dynamic type generation.

### Real-World Cases

**Case 1: React Event Handler Props**
React component libraries use key remapping to generate `onClick`, `onChange`, etc. prop types from event maps.

**Case 2: State Management Selectors**
State management libraries generate selector hooks (`useUserName`, `useUserAge`) from state shapes.

**Case 3: API Client Generation**
API clients generate method names from endpoint definitions, filtering out non-function properties.

**Case 4: Form Field Generation**
Form libraries generate field names with prefixes/suffixes (e.g., `field_` or `_value`) from data types.

**Case 5: ORM Query Builders**
ORMs generate query method names from entity fields, filtering out non-scalar properties.

**Case 6: Type-Safe Event Emitters**
Event emitters generate `on`/`off`/`emit` method names from event maps, ensuring type safety across event handling.

**Case 7: GraphQL Code Generation**
GraphQL generators create query/mutation method names from schema types, filtering based on operation types.

**Case 8: Testing Utilities**
Test utilities generate mock method names from class interfaces, filtering out non-method properties.

### References

- TypeScript 4.1 Release Notes: Key Remapping in Mapped Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#key-remapping-in-mapped-types
- TypeScript Handbook: Mapped Types (Key Remapping via `as`) — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#key-remapping-via-as
- TypeScript PR #40336: Key Remapping in Mapped Types — https://github.com/microsoft/TypeScript/pull/40336
- Remap Keys with `as` Clauses in Mapped Types — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-advanced-patterns/references/tlp-key-remapping-as.md
- Total TypeScript: Key Remapping — https://www.totaltypescript.com/workshops/type-transformations/mapped-types/key-remapping


## Summary: Mapped Type Feature Comparison

| Feature | Syntax | Purpose | TypeScript Version |
|---------|--------|---------|-------------------|
| Key iteration | `[P in keyof T]` | Iterate over all keys | 2.1 |
| Property transformation | `T[P]` | Access/transform property types | 2.1 |
| Add `readonly` | `+readonly` or `readonly` | Make properties immutable | 2.8 |
| Remove `readonly` | `-readonly` | Make properties mutable | 2.8 |
| Add optional | `+?` or `?` | Make properties optional | 2.8 |
| Remove optional | `-?` | Make properties required (and remove `undefined`) | 2.8 |
| Key remapping | `as NewKey` | Rename/filter keys | 4.1 |
| Filter keys | `as ... ? K : never` | Remove properties | 4.1 |
| Exact optional | `exactOptionalPropertyTypes` | Distinguish missing vs `undefined` | 4.4 |
| `number`/`symbol` keys | `keyof` includes these | Support numeric/symbol keys | 2.9 |


## References

- TypeScript Handbook: Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript 2.1 Release Notes: Mapped Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html
- TypeScript 2.8 Release Notes: Improved control over mapped type modifiers — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html
- TypeScript 2.9 Release Notes: Support number and symbol named properties with keyof and mapped types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-9.html
- TypeScript 4.1 Release Notes: Key Remapping in Mapped Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#key-remapping-in-mapped-types
- TypeScript 4.4 Release Notes: exactOptionalPropertyTypes — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html#exact-optional-property-types---exactoptionalpropertytypes
- TypeScript Handbook: Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Handbook: Keyof Type Operator — https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- TypeScript Handbook: Indexed Access Types — https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- TypeScript Playground: Mapped Types — https://www.typescriptlang.org/play/typescript/meta-types/mapped-types.ts.html
- GitHub Issue #60717: How does indexing an optional property within a mapped type behave? — https://github.com/microsoft/TypeScript/issues/60717
- GitHub Issue #60233: Certain homomorphic mappings break assignability with exactOptionalPropertyTypes — https://github.com/microsoft/TypeScript/issues/60233
- GitHub Issue #60138: `exactOptionalPropertyTypes` faults the use of `Omit` — https://github.com/microsoft/TypeScript/issues/60138
- Total TypeScript: Mapped Types — https://www.totaltypescript.com/workshops/type-transformations/mapped-types