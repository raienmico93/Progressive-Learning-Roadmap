# Advanced TypeScript — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced TypeScript refers to the language's higher-order type-system features — conditional types, mapped types, utility types, type guards, advanced generics, decorators, template literal types, and branded types — that enable expressive, reusable, and safe type-level programming.

**Technical Definition:** Advanced TypeScript leverages the type system as a programming language: conditional types (`T extends U ? X : Y`) enable type-level branching, mapped types (`{ [K in keyof T]: ... }`) enable structural transformation, utility types (`Partial`, `Pick`, `Omit`, etc.) provide reusable transformations, type guards (`value is T`) narrow runtime types, generics with constraints and defaults enable flexible abstractions, decorators add metadata and behaviour, template literal types (`\`${A}-${B}\``) enable string-level type manipulation, and branded types (`T & { __brand: 'X' }`) enable nominal typing within a structural system.

**Beginner-Friendly Explanation:** Basic TypeScript tells the compiler what types exist. Advanced TypeScript tells the compiler how to *compute* types. It's like the difference between using a calculator (basic) and writing a program that generates calculators (advanced). Conditional types ask "if this type matches that, then use this type, otherwise that one." Mapped types say "take this type and transform every property." Template literal types say "combine these string types into new string types." Branded types say "this string is not just any string — it's a UserId." These features make TypeScript powerful enough to model complex domains with compile-time guarantees.

### Key Characteristics

- **Type-level programming:** Types are computed, transformed, and composed.
- **Compile-time only:** All advanced type features are erased at runtime.
- **Composable:** Conditional, mapped, and template literal types combine freely.
- **Expressive:** Can model complex domains (state machines, API contracts, DDD).
- **Framework-integrated:** Decorators drive NestJS, TypeORM, Angular.
- **Safety-enhancing:** Branded types, type guards, and exhaustive checks prevent bugs.
- **Complexity trade-off:** Power comes at the cost of readability and compilation time.

### Prerequisites

- **TypeScript fundamentals:** Types, interfaces, generics, unions, intersections.
- **Generics:** Type parameters, constraints, defaults.
- **`keyof` and indexed access:** `keyof T`, `T[K]`.
- **Union and intersection types:** `A | B`, `A & B`.
- **Strict mode:** `strict: true`, `strictNullChecks`, `noImplicitAny`.
- **Familiarity with frameworks:** NestJS, TypeORM, or Angular (for decorators).

### Related Programming Areas

- **Domain-Driven Design (DDD):** Branded types for value objects, aggregate IDs.
- **API design:** Type-safe contracts, OpenAPI generation.
- **ORM/ODM:** Type inference, query builders.
- **Validation:** Zod, TypeBox, class-validator.
- **Testing:** Type-level tests with `tsd`, `expect-type`.
- **Framework internals:** NestJS decorators, TypeORM entities.

### Core Concepts

1. **Conditional Types** — building dynamic type assignments based on generic expressions.
2. **Mapped Types** — transforming existing type structures into new variations.
3. **Utility Types** — leveraging built-in assistants like `Partial`, `Required`, `Readonly`, `Pick`, `Omit`.
4. **Type Guards** — implementing user-defined type guards and `is` assertions to narrow runtime states safely.
5. **Generics** — implementing advanced generic constraints and default values.
6. **Decorators** — leveraging legacy or modern ECMAScript decorators for frameworks like NestJS.
7. **Template Literal Types** — manipulating and combining string types structurally.
8. **Template/Utility Type Branding** — implementing nominal typing / branded types for safe database IDs.

---

## Core Concept 1: Conditional Types

### Definitions

**Core Definition:** Conditional types select one of two types based on whether a type relationship holds, expressed as `T extends U ? X : Y`.

**Technical Definition:** Conditional types in TypeScript follow the syntax `T extends U ? X : Y`, where `T` is checked for assignability to `U`. If `T` is assignable to `U`, the type resolves to `X`; otherwise to `Y`. Conditional types distribute over naked type parameters in unions, enabling powerful type-level computations. The `infer` keyword allows extracting types from within conditional types. Conditional types are the foundation of TypeScript's standard library utilities (`Exclude`, `Extract`, `ReturnType`, `Parameters`, `Awaited`).

**Beginner-Friendly Explanation:** A conditional type is like an if-statement for types. "If T is a string, use `string[]`; otherwise, use `T[]`." The compiler evaluates the condition at compile time and picks the appropriate type. It's how TypeScript can "compute" types based on other types.

### Purposes

- To select types dynamically based on other types.
- To extract types using `infer`.
- To filter unions (`Exclude`, `Extract`).
- To unwrap types (`Awaited`, `ReturnType`).
- To build reusable type transformations.

### Syntax Rules and Structure

#### Basic Conditional Type

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<'hello'>; // true
type B = IsString<42>;       // false
```

#### Conditional Type with `infer`

```typescript
type ElementType<T> = T extends (infer U)[] ? U : never;

type A = ElementType<string[]>;  // string
type B = ElementType<number[]>;  // number
type C = ElementType<boolean>;   // never

type ReturnTypeOf<T> = T extends (...args: any[]) => infer R ? R : never;

type D = ReturnTypeOf<() => string>;            // string
type E = ReturnTypeOf<(a: number) => boolean>;  // boolean
```

#### Distributive Conditional Types

```typescript
type ToArray<T> = T extends any ? T[] : never;

type A = ToArray<string | number>;
// Distributes: string[] | number[]
// NOT (string | number)[]

// Prevent distribution with a tuple
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;
type B = ToArrayNonDist<string | number>;
// (string | number)[]
```

#### Standard Utility Implementations

```typescript
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;
type NonNullable<T> = T extends null | undefined ? never : T;
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;
type Parameters<T> = T extends (...args: infer P) => any ? P : never;
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type ConstructorParameters<T> = T extends abstract new (...args: infer P) => any ? P : never;
type InstanceType<T> = T extends abstract new (...args: any) => infer R ? R : never;
```

#### Syntax Rules

- **Use `T extends U ? X : Y`** — the basic syntax.
- **Use `infer` to extract types** — `T extends Promise<infer U> ? U : T`.
- **Be aware of distribution** — naked type parameters distribute over unions.
- **Use `[T] extends [U]`** — to prevent distribution.
- **Use `never` for impossible branches** — `T extends string ? T : never`.
- **Recursive conditional types** — supported for deep unwrapping.
- **Combine with mapped types** — for advanced transformations.

#### Constraints and Limitations

- **Distribution can be surprising** — naked type parameters distribute.
- **Recursion depth** — TypeScript limits recursion (typically 50 levels).
- **Compilation time** — complex conditional types slow down the compiler.
- **Readability** — deeply nested conditionals are hard to understand.
- **No runtime effect** — conditional types are erased.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Type-Safe API Response Unwrapping

```typescript
// conditional-types.ts
// Unwrap a Promise recursively
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;

type A = Awaited<Promise<string>>;              // string
type B = Awaited<Promise<Promise<number>>>;     // number
type C = Awaited<string>;                        // string

// Extract the data type from an API response
type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: string };

type UnwrapResponse<T> = T extends { success: true; data: infer D } ? D : never;

type UserResponse = ApiResponse<{ id: string; name: string }>;
type UserData = UnwrapResponse<UserResponse>;
// { id: string; name: string }

// Function to extract data
function unwrap<T>(response: ApiResponse<T>): T {
  if (response.success) {
    return response.data;
  }
  throw new Error(response.error);
}

const response: ApiResponse<number> = { success: true, data: 42 };
const data = unwrap(response);
console.log(data); // 42
```

**Expected Output:**
```
42
```

**Why this works:** The `Awaited` type recursively unwraps Promises. `UnwrapResponse` extracts the `data` field from a successful response. The `unwrap` function returns a typed value based on the response shape.

### Real-World Cases

- **API clients:** Unwrapping `ApiResponse<T>` to `T`.
- **ORM queries:** Extracting result types from query builders.
- **React hooks:** Extracting state types from reducers.
- **Library APIs:** `ReturnType`, `Parameters`, `Awaited`.

---

## Core Concept 2: Mapped Types

### Definitions

**Core Definition:** Mapped types transform existing types by iterating over their keys and producing new types with modified properties.

**Technical Definition:** Mapped types use the syntax `{ [K in keyof T]: NewType }` to iterate over the keys of `T` and produce a new type. Modifiers `+?`/`-?` (optionality) and `+readonly`/`-readonly` (mutability) can be added or removed. The `as` clause enables key remapping (`{ [K in keyof T as NewKey]: ... }`). Mapped types are the foundation of `Partial`, `Required`, `Readonly`, `Pick`, and `Omit`.

**Beginner-Friendly Explanation:** A mapped type is like a photocopier with a transformation option. It takes a type, walks through every property, and produces a new type with modified properties — making them optional, readonly, or renamed. It's how you can say "give me the same type, but all properties are optional."

### Purposes

- To transform all properties of a type.
- To add or remove modifiers (`?`, `readonly`).
- To remap keys with the `as` clause.
- To create variants of existing types.
- To build utility types.

### Syntax Rules and Structure

#### Basic Mapped Type

```typescript
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

type MyReadonly<T> = {
  readonly [K in keyof T]: T[K];
};

type MyRequired<T> = {
  [K in keyof T]-?: T[K];
};

type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};
```

#### Key Remapping with `as`

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// {
//   getName: () => string;
//   getAge: () => number;
// }
```

#### Filtering Keys

```typescript
type OmitByType<T, U> = {
  [K in keyof T as T[K] extends U ? never : K]: T[K];
};

interface Mixed {
  name: string;
  age: number;
  isActive: boolean;
  email: string;
}

type OnlyStrings = OmitByType<Mixed, string>;
// { age: number; isActive: boolean }
```

#### Syntax Rules

- **Use `[K in keyof T]`** — the basic syntax.
- **Use `?:` to add optionality** — `[K in keyof T]?: T[K]`.
- **Use `-?:` to remove optionality** — `[K in keyof T]-?: T[K]`.
- **Use `readonly` and `-readonly`** — to add/remove mutability.
- **Use `as` for key remapping** — `[K in keyof T as NewKey]`.
- **Use `string & K`** — to ensure `K` is a string for `Capitalize`.
- **Combine with conditional types** — for filtering.

#### Constraints and Limitations

- **Mapped types are erased** — no runtime effect.
- **Key remapping requires string keys** — `string & K` for template literals.
- **Homomorphic vs. non-homomorphic** — homomorphic mapped types preserve modifiers.
- **Complexity** — deeply nested mapped types are hard to read.
- **Compilation time** — large mapped types slow down the compiler.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Building Utility Types with Mapped Types

```typescript
// mapped-types.ts
// Recreate standard utility types
type MyPartial<T> = { [K in keyof T]?: T[K] };
type MyRequired<T> = { [K in keyof T]-?: T[K] };
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };
type MyPick<T, K extends keyof T> = { [P in K]: T[P] };
type MyOmit<T, K extends keyof T> = { [P in Exclude<keyof T, K>]: T[P] };
type MyRecord<K extends keyof any, T> = { [P in K]: T };

// Key remapping — create setters
type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

interface User {
  name: string;
  age: number;
  email: string;
}

type UserSetters = Setters<User>;
// {
//   setName: (value: string) => void;
//   setAge: (value: number) => void;
//   setEmail: (value: string) => void;
// }

// Filter by value type
type FilterByValueType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

type StringFields = FilterByValueType<User, string>;
// { name: string; email: string }

// Deep partial
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

type PartialUser = DeepPartial<User>;
// { name?: string; age?: number; email?: string }

console.log('Mapped types are compile-time only');
```

**Expected Output:**
```
Mapped types are compile-time only
```

**Why this works:** Mapped types iterate over keys and transform properties. Key remapping (`as`) creates new key names. Filtering with `never` removes keys. Recursive mapped types (`DeepPartial`) handle nested objects.

### Real-World Cases

- **API DTOs:** `Partial<T>` for PATCH requests.
- **Form state:** `Readonly<T>` for immutable state.
- **ORM entities:** `Omit<T, 'passwordHash'>` for safe responses.
- **Redux reducers:** Mapped types for action creators.

---

## Core Concept 3: Utility Types

### Definitions

**Core Definition:** Utility types are built-in generic types that perform common type transformations, such as making properties optional, picking a subset, or omitting keys.

**Technical Definition:** TypeScript provides a standard library of utility types: `Partial<T>`, `Required<T>`, `Readonly<T>`, `Pick<T, K>`, `Omit<T, K>`, `Record<K, V>`, `Exclude<T, U>`, `Extract<T, U>`, `NonNullable<T>`, `Parameters<T>`, `ReturnType<T>`, `ConstructorParameters<T>`, `InstanceType<T>`, `Awaited<T>`, `ThisParameterType<T>`, `OmitThisParameter<T>`, `ThisType<T>`, `Uppercase<T>`, `Lowercase<T>`, `Capitalize<T>`, `Uncapitalize<T>`. They are implemented with mapped and conditional types and can be composed.

**Beginner-Friendly Explanation:** Utility types are pre-built type transformations — like tools in a toolbox. Instead of writing your own `Partial` type, TypeScript gives you one. Need to make all properties optional? Use `Partial`. Need to remove a field? Use `Omit`. Need to pick specific fields? Use `Pick`. They save time and reduce errors.

### Purposes

- To avoid reinventing common type transformations.
- To compose complex types from simple ones.
- To improve code readability.
- To ensure consistency across the codebase.
- To enable type-safe refactoring.

### Syntax Rules and Structure

#### Object Utility Types

| Utility | Purpose | Example |
|---------|---------|---------|
| `Partial<T>` | All properties optional | `Partial<User>` |
| `Required<T>` | All properties required | `Required<User>` |
| `Readonly<T>` | All properties readonly | `Readonly<User>` |
| `Pick<T, K>` | Subset of properties | `Pick<User, 'id' \| 'name'>` |
| `Omit<T, K>` | All except K | `Omit<User, 'password'>` |
| `Record<K, V>` | Object with keys K and values V | `Record<string, number>` |

#### Union Utility Types

| Utility | Purpose | Example |
|---------|---------|---------|
| `Exclude<T, U>` | Remove U from T | `Exclude<'a' \| 'b', 'a'>` → `'b'` |
| `Extract<T, U>` | Keep only U from T | `Extract<'a' \| 'b', 'a'>` → `'a'` |
| `NonNullable<T>` | Remove null/undefined | `NonNullable<string \| null>` → `string` |

#### Function Utility Types

| Utility | Purpose | Example |
|---------|---------|---------|
| `Parameters<T>` | Parameter types | `Parameters<(a: number) => void>` → `[number]` |
| `ReturnType<T>` | Return type | `ReturnType<() => string>` → `string` |
| `ConstructorParameters<T>` | Constructor params | `ConstructorParameters<typeof Date>` |
| `InstanceType<T>` | Instance type | `InstanceType<typeof Date>` → `Date` |
| `Awaited<T>` | Unwrap Promise | `Awaited<Promise<string>>` → `string` |

#### String Utility Types

| Utility | Purpose | Example |
|---------|---------|---------|
| `Uppercase<T>` | Uppercase string literal | `Uppercase<'hello'>` → `'HELLO'` |
| `Lowercase<T>` | Lowercase string literal | `Lowercase<'HELLO'>` → `'hello'` |
| `Capitalize<T>` | Capitalise first letter | `Capitalize<'hello'>` → `'Hello'` |
| `Uncapitalize<T>` | Uncapitalise first letter | `Uncapitalize<'Hello'>` → `'hello'` |

#### Syntax Rules

- **Use `Partial<T>` for PATCH DTOs** — all fields optional.
- **Use `Omit<T, 'password'>` for safe responses** — remove sensitive fields.
- **Use `Pick<T, K>` for subsets** — only the needed fields.
- **Use `Record<K, V>` for dictionaries** — typed maps.
- **Use `ReturnType<T>` for function return types** — avoid duplication.
- **Use `Awaited<T>` for unwrapping Promises** — in async contexts.
- **Compose utility types** — `Partial<Pick<T, K>>`.

#### Constraints and Limitations

- **Utility types are erased** — no runtime effect.
- **`Omit` is not strict** — it does not check if K exists in T.
- **`Pick` requires K extends keyof T** — type-safe.
- **Some utilities lose modifiers** — `Pick` preserves modifiers, `Omit` preserves modifiers.
- **Composition can be verbose** — `Partial<Omit<T, 'id'>>`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Composing Utility Types for API DTOs

```typescript
// utility-types.ts
interface User {
  id: string;
  email: string;
  name: string;
  passwordHash: string;
  role: 'user' | 'editor' | 'admin';
  createdAt: Date;
  updatedAt: Date;
}

// Input DTO for creation — omit generated fields
type CreateUserDto = Omit<User, 'id' | 'createdAt' | 'updatedAt' | 'passwordHash'> & {
  password: string;
};

// Input DTO for update — all fields optional except id
type UpdateUserDto = Partial<Omit<User, 'id' | 'createdAt' | 'updatedAt'>>;

// Output DTO — omit sensitive fields
type UserDto = Omit<User, 'passwordHash'>;

// Query DTO — filters and pagination
type ListUsersQuery = Partial<Pick<User, 'role'>> & {
  page?: number;
  limit?: number;
  search?: string;
};

// Record for role permissions
type RolePermissions = Record<User['role'], string[]>;

const permissions: RolePermissions = {
  user: ['read:own'],
  editor: ['read:any', 'write:any'],
  admin: ['read:any', 'write:any', 'delete:any'],
};

// Usage
const userDto: UserDto = {
  id: 'user-1',
  email: 'alice@example.com',
  name: 'Alice',
  role: 'admin',
  createdAt: new Date(),
  updatedAt: new Date(),
};

console.log(userDto.email);       // 'alice@example.com'
console.log(permissions.admin);   // ['read:any', 'write:any', 'delete:any']
```

**Expected Output:**
```
alice@example.com
[ 'read:any', 'write:any', 'delete:any' ]
```

**Why this works:** Utility types compose to create precise DTOs. `Omit` removes sensitive fields. `Partial` makes fields optional. `Pick` selects specific fields. `Record` types the permissions map.

### Real-World Cases

- **REST APIs:** Input/output DTOs with `Omit`, `Pick`, `Partial`.
- **React:** `PropsWithChildren`, `ComponentProps`, `ReturnType`.
- **Redux:** `ReturnType<typeof reducer>` for state types.
- **ORM:** `Omit<User, 'passwordHash'>` for safe responses.

---

## Core Concept 4: Type Guards

### Definitions

**Core Definition:** A type guard is a runtime check that narrows a value's type within a conditional block, expressed with `typeof`, `instanceof`, `in`, or user-defined `is` assertions.

**Technical Definition:** Type guards narrow the type of a value within a conditional branch. Built-in guards include `typeof` (primitives), `instanceof` (classes), `in` (property presence), and truthiness checks. User-defined type guards use the `value is Type` return type, which tells TypeScript that the function returning `true` means the value is of that type. Assertion functions (`asserts value is Type`) throw if the condition is false, narrowing the type for the rest of the scope.

**Beginner-Friendly Explanation:** A type guard is like a bouncer checking IDs. "If this person is over 18, let them in." TypeScript uses the check to narrow the type — inside the `if` block, the value is treated as the narrowed type. User-defined guards let you teach TypeScript about custom checks.

### Purposes

- To narrow union types safely.
- To validate runtime data.
- To enable type-safe access to type-specific members.
- To replace type assertions (`as`) with safe checks.
- To implement exhaustive checks with `never`.

### Syntax Rules and Structure

#### Built-in Type Guards

```typescript
function format(value: string | number | boolean): string {
  if (typeof value === 'string') return value.toUpperCase();
  if (typeof value === 'number') return value.toFixed(2);
  return value ? 'yes' : 'no'; // boolean
}

class Animal { name = 'animal'; }
class Dog extends Animal { breed = 'lab'; }

function describe(animal: Animal): string {
  if (animal instanceof Dog) return `Dog: ${animal.breed}`;
  return `Animal: ${animal.name}`;
}

interface Admin { role: 'admin'; permissions: string[]; }
interface User { role: 'user'; email: string; }

function greet(person: Admin | User): string {
  if ('permissions' in person) return `Admin with ${person.permissions.length} permissions`;
  return `User: ${person.email}`;
}
```

#### User-Defined Type Guard

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    typeof (value as User).id === 'string' &&
    'name' in value &&
    typeof (value as User).name === 'string' &&
    'email' in value &&
    typeof (value as User).email === 'string'
  );
}

function processValue(value: unknown): void {
  if (isUser(value)) {
    console.log(value.name); // value is User
  } else {
    console.log('Not a user');
  }
}
```

#### Assertion Function

```typescript
function assertIsUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error('Not a user');
  }
}

function handleValue(value: unknown): void {
  assertIsUser(value);
  // value is now User for the rest of the scope
  console.log(value.email);
}
```

#### Discriminated Union Guard

```typescript
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number }
  | { kind: 'rectangle'; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle': return Math.PI * shape.radius ** 2;
    case 'square': return shape.side ** 2;
    case 'rectangle': return shape.width * shape.height;
    default:
      const _exhaustive: never = shape;
      throw new Error(`Unhandled shape: ${JSON.stringify(_exhaustive)}`);
  }
}
```

#### Syntax Rules

- **Use `typeof` for primitives** — `typeof value === 'string'`.
- **Use `instanceof` for classes** — `value instanceof Date`.
- **Use `in` for property presence** — `'role' in value`.
- **Use `value is Type`** — for user-defined guards.
- **Use `asserts value is Type`** — for assertion functions.
- **Use discriminated unions** — with a `kind` or `type` field.
- **Use `never` for exhaustiveness** — in `default` branches.
- **Validate deeply** — check all required properties.
- **Avoid `as` assertions** — use type guards instead.

#### Constraints and Limitations

- **User-defined guards can lie** — TypeScript trusts the return type.
- **Assertion functions must throw** — otherwise the type is wrong.
- **`in` only checks property presence** — not the type of the value.
- **`instanceof` fails across realms** — iframes, workers.
- **Type guards add runtime checks** — small performance cost.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Type Guards for API Response Parsing

```typescript
// type-guards.ts
interface SuccessResponse<T> {
  success: true;
  data: T;
}

interface ErrorResponse {
  success: false;
  error: string;
  code: string;
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function isSuccess<T>(response: ApiResponse<T>): response is SuccessResponse<T> {
  return response.success === true;
}

function isError<T>(response: ApiResponse<T>): response is ErrorResponse {
  return response.success === false;
}

function handleResponse<T>(response: ApiResponse<T>): T | never {
  if (isSuccess(response)) {
    return response.data;
  }
  if (isError(response)) {
    throw new Error(`${response.code}: ${response.error}`);
  }
  const _exhaustive: never = response;
  throw new Error(`Unhandled response: ${JSON.stringify(_exhaustive)}`);
}

// User-defined guard for unknown data
interface User {
  id: string;
  name: string;
  email: string;
}

function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    typeof value.id === 'string' &&
    'name' in value &&
    typeof value.name === 'string' &&
    'email' in value &&
    typeof value.email === 'string'
  );
}

// Usage
const response: ApiResponse<User> = {
  success: true,
  data: { id: 'user-1', name: 'Alice', email: 'alice@example.com' },
};

const data = handleResponse(response);
console.log(data.name); // 'Alice'

const unknownData: unknown = { id: 'user-2', name: 'Bob', email: 'bob@example.com' };
if (isUser(unknownData)) {
  console.log(unknownData.email); // 'bob@example.com'
}
```

**Expected Output:**
```
Alice
bob@example.com
```

**Why this works:** `isSuccess` and `isError` narrow the `ApiResponse` union. `handleResponse` returns the typed data or throws. `isUser` validates unknown data at runtime and narrows the type.

### Real-World Cases

- **API responses:** Distinguishing success from error responses.
- **Form validation:** Validating and narrowing form data.
- **Parser development:** Type-safe parsing of JSON, CSV, XML.
- **Event handling:** Narrowing event types.

---

## Core Concept 5: Advanced Generics

### Definitions

**Core Definition:** Advanced generics use constraints, defaults, and multiple type parameters to build flexible, reusable abstractions while preserving type safety.

**Technical Definition:** Advanced generics include: **constraints** (`<T extends U>`) that restrict type parameters; **defaults** (`<T = string>`) that provide fallbacks; **multiple parameters** (`<K, V>`) for key-value relationships; **`keyof` constraints** (`<T, K extends keyof T>`) for property access; **conditional generics** (`<T extends string ? A : B>`) for type-level branching; **variadic tuple types** (`<T extends any[]>`) for function composition; **const type parameters** (`<const T>`) for literal inference; **generic constraints with `infer`** for extracting types.

**Beginner-Friendly Explanation:** Advanced generics are like a Swiss Army knife for types. Basic generics say "this works with any type." Advanced generics say "this works with any type that has a `length` property" or "this works with any type, but defaults to `string` if not specified." They give you fine-grained control over what types are allowed and how they behave.

### Purposes

- To restrict type parameters to compatible types.
- To provide default types when none are specified.
- To relate multiple type parameters.
- To extract types from complex structures.
- To build reusable, type-safe abstractions.

### Syntax Rules and Structure

#### Constraints

```typescript
// T must have a length property
function logLength<T extends { length: number }>(item: T): void {
  console.log(item.length);
}

logLength('hello');       // OK
logLength([1, 2, 3]);     // OK
logLength({ length: 10 }); // OK

// K must be a key of T
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'Alice', age: 30 };
const name = getProperty(user, 'name'); // string
const age = getProperty(user, 'age');   // number
```

#### Defaults

```typescript
interface ApiResponse<T = unknown, E = Error> {
  data?: T;
  error?: E;
  status: number;
}

const response1: ApiResponse = { status: 200 };
const response2: ApiResponse<string> = { status: 200, data: 'hello' };
const response3: ApiResponse<string, TypeError> = { status: 500, error: new TypeError() };
```

#### Multiple Parameters

```typescript
function mapObject<T, U>(
  obj: Record<string, T>,
  fn: (value: T) => U,
): Record<string, U> {
  const result: Record<string, U> = {};
  for (const [key, value] of Object.entries(obj)) {
    result[key] = fn(value);
  }
  return result;
}

const numbers = { a: 1, b: 2, c: 3 };
const strings = mapObject(numbers, (n) => n.toString());
// { a: '1', b: '2', c: '3' }
```

#### Const Type Parameters

```typescript
// Without const — T is widened to string[]
function tuple1<T extends readonly unknown[]>(items: T): T {
  return items;
}
const a = tuple1(['a', 'b']); // string[]

// With const — T is inferred as readonly ['a', 'b']
function tuple2<const T extends readonly unknown[]>(items: T): T {
  return items;
}
const b = tuple2(['a', 'b']); // readonly ['a', 'b']
```

#### Variadic Tuple Types

```typescript
function concat<T extends unknown[], U extends unknown[]>(a: T, b: U): [...T, ...U] {
  return [...a, ...b];
}

const result = concat([1, 2] as const, ['a', 'b'] as const);
// readonly [1, 2, 'a', 'b']
```

#### Syntax Rules

- **Use `extends` for constraints** — `<T extends U>`.
- **Use `=` for defaults** — `<T = string>`.
- **Use `keyof T` for property keys** — `<K extends keyof T>`.
- **Use `const` for literal inference** — `<const T>`.
- **Use variadic tuples** — `<T extends unknown[]>`.
- **Use `infer` in constraints** — `<T extends Promise<infer U>>`.
- **Use multiple parameters** — `<K, V>`.
- **Document constraints** — with JSDoc.

#### Constraints and Limitations

- **Complex constraints are hard to read** — break into smaller types.
- **Defaults do not constrain** — a default must satisfy the constraint.
- **Const type parameters** — TypeScript 5.0+.
- **Variadic tuples** — TypeScript 4.0+.
- **Inference may fail** — explicit annotations may be needed.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Advanced Generic Repository

```typescript
// advanced-generics.ts
interface Entity {
  id: string;
}

interface Repository<T extends Entity> {
  findById(id: string): Promise<T | null>;
  findWhere<K extends keyof T>(key: K, value: T[K]): Promise<T[]>;
  create(data: Omit<T, 'id'>): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
}

class InMemoryRepository<T extends Entity> implements Repository<T> {
  private items = new Map<string, T>();

  async findById(id: string): Promise<T | null> {
    return this.items.get(id) ?? null;
  }

  async findWhere<K extends keyof T>(key: K, value: T[K]): Promise<T[]> {
    return [...this.items.values()].filter((item) => item[key] === value);
  }

  async create(data: Omit<T, 'id'>): Promise<T> {
    const id = crypto.randomUUID();
    const entity = { ...data, id } as T;
    this.items.set(id, entity);
    return entity;
  }

  async update(id: string, data: Partial<T>): Promise<T> {
    const existing = this.items.get(id);
    if (!existing) throw new Error('Not found');
    const updated = { ...existing, ...data };
    this.items.set(id, updated);
    return updated;
  }
}

interface User extends Entity {
  name: string;
  email: string;
  role: 'user' | 'admin';
}

(async () => {
  const repo = new InMemoryRepository<User>();

  const user = await repo.create({
    name: 'Alice',
    email: 'alice@example.com',
    role: 'admin',
  });

  console.log(user.id);    // UUID
  console.log(user.name);  // 'Alice'

  const admins = await repo.findWhere('role', 'admin');
  console.log(admins.length); // 1
})();
```

**Expected Output:**
```
<uuid>
Alice
1
```

**Why this works:** The `Repository<T extends Entity>` interface constrains `T` to entities with an `id`. `findWhere` uses `K extends keyof T` to ensure the key exists. `create` accepts `Omit<T, 'id'>`. `update` accepts `Partial<T>`. All operations are type-safe.

### Real-World Cases

- **Repositories:** Generic CRUD with typed entities.
- **API clients:** Generic HTTP methods with typed responses.
- **Utility functions:** `map`, `filter`, `groupBy`, `pluck`.
- **Form libraries:** Generic form state and validation.

---

## Core Concept 6: Decorators

### Definitions

**Core Definition:** Decorators are functions that add metadata or behaviour to classes, methods, properties, or parameters, used heavily by frameworks like NestJS, TypeORM, and Angular.

**Technical Definition:** Decorators are a stage 3 ECMAScript proposal with two variants: **legacy decorators** (TypeScript's experimental implementation, enabled with `experimentalDecorators: true`) and **standard decorators** (TypeScript 5.0+, following the TC39 proposal). Legacy decorators receive `(target, propertyKey, descriptor)` and can modify behaviour via `Object.defineProperty`. Standard decorators receive `(value, context)` and return a replacement value. NestJS, TypeORM, and Angular use legacy decorators. Frameworks use decorators for dependency injection (`@Injectable`), routing (`@Controller`, `@Get`), validation (`@IsString`), and ORM mapping (`@Entity`, `@Column`).

**Beginner-Friendly Explanation:** A decorator is like a sticky note you attach to a class or method. The note says "this class should be injectable" or "this method handles GET requests." The framework reads the sticky notes and does the right thing. Decorators are how frameworks like NestJS know how to wire up your application.

### Purposes

- To add metadata to classes, methods, and properties.
- To enable dependency injection.
- To define routes and HTTP methods.
- To validate data.
- To map ORM entities.
- To implement cross-cutting concerns (logging, caching).

### Syntax Rules and Structure

#### Legacy Decorators (TypeScript)

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

```typescript
// Class decorator
function Injectable(): ClassDecorator {
  return (target) => {
    Reflect.defineMetadata('injectable', true, target);
  };
}

// Method decorator
function Log(): MethodDecorator {
  return (target, propertyKey, descriptor: PropertyDescriptor) => {
    const original = descriptor.value;
    descriptor.value = function (...args: unknown[]) {
      console.log(`Calling ${String(propertyKey)} with`, args);
      return original.apply(this, args);
    };
  };
}

// Property decorator
function Column(): PropertyDecorator {
  return (target, propertyKey) => {
    Reflect.defineMetadata('column', true, target, propertyKey);
  };
}

// Parameter decorator
function Inject(token: string): ParameterDecorator {
  return (target, propertyKey, parameterIndex) => {
    Reflect.defineMetadata('inject', token, target, propertyKey, parameterIndex);
  };
}

// Usage
@Injectable()
class UserService {
  @Column()
  name: string;

  constructor(@Inject('DATABASE') private db: Database) {}

  @Log()
  findById(id: string): User {
    return this.db.query(id);
  }
}
```

#### Standard Decorators (TypeScript 5.0+)

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": false // Standard decorators
  }
}
```

```typescript
// Class decorator (standard)
function logged<T extends new (...args: any[]) => any>(
  value: T,
  context: ClassDecoratorContext<T>,
): T {
  return class extends value {
    constructor(...args: any[]) {
      super(...args);
      console.log(`Created ${context.name}`);
    }
  };
}

// Method decorator (standard)
function log<This, Args extends any[], Return>(
  value: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>,
): (this: This, ...args: Args) => Return {
  return function (this: This, ...args: Args): Return {
    console.log(`Calling ${String(context.name)} with`, args);
    return value.apply(this, args);
  };
}

@logged
class UserService {
  @log
  findById(id: string): string {
    return `User ${id}`;
  }
}
```

#### NestJS Example

```typescript
import { Injectable, Controller, Get, Param, Inject } from '@nestjs/common';

@Injectable()
export class UserService {
  constructor(
    @Inject('USER_REPOSITORY') private readonly userRepository: UserRepository,
  ) {}

  async findById(id: string): Promise<UserDto> {
    const user = await this.userRepository.findById(id);
    if (!user) throw new NotFoundException('User not found');
    return UserMapper.toDto(user);
  }
}

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  async findById(@Param('id') id: string): Promise<UserDto> {
    return this.userService.findById(id);
  }
}

@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;
}
```

#### Syntax Rules

- **Enable `experimentalDecorators` for legacy** — NestJS, TypeORM.
- **Use standard decorators for new code** — TypeScript 5.0+.
- **Use `@Injectable()` for services** — NestJS dependency injection.
- **Use `@Controller()` and `@Get()` for routes** — NestJS.
- **Use `@Entity()` and `@Column()` for ORM** — TypeORM.
- **Use `@IsString()` for validation** — class-validator.
- **Keep decorators focused** — one responsibility per decorator.
- **Document custom decorators** — with JSDoc.

#### Constraints and Limitations

- **Legacy decorators are experimental** — not standardised.
- **Standard decorators are newer** — less framework support.
- **Decorators are metadata-only** — no runtime type information.
- **`emitDecoratorMetadata`** — required for type-based DI.
- **Mixing legacy and standard** — not possible in the same project.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom NestJS Decorators

```typescript
// decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export interface AuthUser {
  id: string;
  email: string;
  roles: string[];
}

export const CurrentUser = createParamDecorator(
  (data: keyof AuthUser | undefined, ctx: ExecutionContext): AuthUser | AuthUser[keyof AuthUser] => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user as AuthUser;

    return data ? user[data] : user;
  },
);
```

```typescript
// decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);
```

```typescript
// guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles || requiredRoles.length === 0) return true;

    const request = context.switchToHttp().getRequest();
    const user = request.user as AuthUser;

    const hasRole = requiredRoles.some((role) => user.roles.includes(role));
    if (!hasRole) throw new ForbiddenException('Insufficient role');
    return true;
  }
}
```

```typescript
// controllers/admin.controller.ts
@Controller('admin')
@UseGuards(RolesGuard)
@Roles('admin')
export class AdminController {
  @Get('users')
  async listUsers(@CurrentUser() user: AuthUser): Promise<UserDto[]> {
    console.log(`Admin ${user.email} listed users`);
    return this.usersService.findAll();
  }
}
```

**Expected behaviour:** `@CurrentUser()` injects the authenticated user into controller methods. `@Roles('admin')` restricts access. The `RolesGuard` checks the user's roles. Requests without the required role return `403`.

**Why this works:** Custom decorators add metadata (`SetMetadata`) and inject request data (`createParamDecorator`). The guard reads the metadata with `Reflector` and enforces the rules.

### Real-World Cases

- **NestJS:** `@Controller`, `@Get`, `@Injectable`, `@Inject`.
- **TypeORM:** `@Entity`, `@Column`, `@PrimaryGeneratedColumn`.
- **class-validator:** `@IsString`, `@IsEmail`, `@MinLength`.
- **Angular:** `@Component`, `@Input`, `@Output`.

---

## Core Concept 7: Template Literal Types

### Definitions

**Core Definition:** Template literal types use the syntax of template literals to construct and manipulate string types at compile time.

**Technical Definition:** Template literal types (TypeScript 4.1+) combine string literal types using the syntax `` `${A}${B}` ``. They support built-in string manipulation utilities: `Uppercase<T>`, `Lowercase<T>`, `Capitalize<T>`, `Uncapitalize<T>`. They can be used with `infer` to extract parts of strings, in mapped types for key remapping, and in conditional types for pattern matching. They enable precise typing of string-based APIs (event names, CSS properties, route paths, colour codes).

**Beginner-Friendly Explanation:** Template literal types are like string templates but for types. Instead of `` `Hello, ${name}` `` producing a string at runtime, `` `Hello, ${Name}` `` produces a type at compile time. If `Name` is `'Alice' | 'Bob'`, the result is `'Hello, Alice' | 'Hello, Bob'`. They let you compute string types the same way you compute string values.

### Purposes

- To construct string types from other types.
- To enforce naming conventions (e.g., `get${Capitalize<K>}`).
- To extract parts of strings with `infer`.
- To type event names, routes, and CSS properties.
- To enable precise autocomplete for string-based APIs.

### Syntax Rules and Structure

#### Basic Template Literal Type

```typescript
type Greeting = `Hello, ${string}`;

const a: Greeting = 'Hello, Alice';  // OK
const b: Greeting = 'Hello, Bob';    // OK
// const c: Greeting = 'Hi, Alice'; // Error
```

#### Union Distribution

```typescript
type Size = 'small' | 'medium' | 'large';
type Color = 'red' | 'blue';

type Variant = `${Size}-${Color}`;
// 'small-red' | 'small-blue' | 'medium-red' | 'medium-blue' | 'large-red' | 'large-blue'
```

#### String Manipulation Utilities

```typescript
type A = Uppercase<'hello'>;       // 'HELLO'
type B = Lowercase<'HELLO'>;       // 'hello'
type C = Capitalize<'hello'>;      // 'Hello'
type D = Uncapitalize<'Hello'>;    // 'hello'

type EventName = `on${Capitalize<'click' | 'hover' | 'focus'>}`;
// 'onClick' | 'onHover' | 'onFocus'
```

#### Extracting with `infer`

```typescript
type ExtractRouteParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractRouteParams<`/${Rest}`>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

type Params = ExtractRouteParams<'/users/:userId/posts/:postId'>;
// 'userId' | 'postId'
```

#### Mapped Types with Template Literals

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }
```

#### Syntax Rules

- **Use `` `${A}${B}` ``** — combine string types.
- **Use `string`, `number`, `boolean`, `bigint`** — as placeholders.
- **Use built-in utilities** — `Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`.
- **Use `infer` to extract** — from template literal patterns.
- **Use with mapped types** — for key remapping.
- **Use with conditional types** — for pattern matching.
- **Distribute over unions** — template literals distribute automatically.
- **Combine with `as const`** — for literal inference.

#### Constraints and Limitations

- **Compile-time only** — no runtime effect.
- **Complex patterns are slow** — compilation time increases.
- **Recursion depth** — limited (typically 50 levels).
- **No regex** — template literals are not regular expressions.
- **Limited to strings** — numbers, booleans, bigints, null, undefined are allowed.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Type-Safe Event Emitter

```typescript
// template-literal-types.ts
// Event map
interface EventMap {
  userCreated: { id: string; email: string };
  userUpdated: { id: string; changes: string[] };
  userDeleted: { id: string };
  postCreated: { id: string; authorId: string };
}

// Type-safe event names
type EventName = keyof EventMap;
type EventHandler<T extends EventName> = (payload: EventMap[T]) => void;

class TypedEventEmitter {
  private handlers = new Map<EventName, Set<EventHandler<EventName>>>();

  on<T extends EventName>(event: T, handler: EventHandler<T>): void {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, new Set());
    }
    this.handlers.get(event)!.add(handler as EventHandler<EventName>);
  }

  emit<T extends EventName>(event: T, payload: EventMap[T]): void {
    const handlers = this.handlers.get(event);
    if (handlers) {
      for (const handler of handlers) {
        handler(payload);
      }
    }
  }
}

// Route parameter extraction
type ExtractParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractParams<`/${Rest}`>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

type RouteParams = ExtractParams<'/users/:userId/posts/:postId'>;
// 'userId' | 'postId'

// Usage
const emitter = new TypedEventEmitter();

emitter.on('userCreated', (payload) => {
  console.log(`User created: ${payload.email}`);
  // payload is { id: string; email: string }
});

emitter.emit('userCreated', { id: 'user-1', email: 'alice@example.com' });
// 'User created: alice@example.com'

// CSS property typing
type CSSUnit = 'px' | 'rem' | 'em' | '%';
type CSSValue = `${number}${CSSUnit}`;

const width: CSSValue = '100px';    // OK
const height: CSSValue = '2.5rem';  // OK
// const bad: CSSValue = '100';     // Error

console.log(width, height);
```

**Expected Output:**
```
User created: alice@example.com
100px 2.5rem
```

**Why this works:** Template literal types construct precise string types. `EventName` is constrained to the keys of `EventMap`. `ExtractParams` extracts route parameters. `CSSValue` enforces the format `number + unit`.

### Real-World Cases

- **Event emitters:** Type-safe event names and payloads.
- **Routing:** Extracting parameters from route patterns.
- **CSS-in-JS:** Typing CSS values with units.
- **API paths:** Type-safe URL construction.
- **i18n:** Typing translation keys.

---

## Core Concept 8: Template/Utility Type Branding (Nominal Typing)

### Definitions

**Core Definition:** Branded types (nominal typing) use intersection types with a unique marker to distinguish structurally identical types, such as `UserId` vs. `PostId` — both strings, but not interchangeable.

**Technical Definition:** TypeScript's type system is structural — two types with the same shape are interchangeable. Branded types break this by intersecting a base type with a unique marker (`type UserId = string & { readonly __brand: 'UserId' }`). The brand is a phantom type — it exists only at compile time. Values are created via type assertions (`as UserId`) or constructor functions (`createUserId(value: string): UserId`). Brands can be combined with template literal types for additional constraints. The pattern is also called "nominal typing," "opaque types," or "tagged types."

**Beginner-Friendly Explanation:** Imagine two identical-looking keys — one opens the front door, one opens the back door. Structurally, they're the same. But you don't want to use the front-door key on the back door. Branded types are like labelling the keys: `FrontDoorKey` and `BackDoorKey`. Even though both are just metal, TypeScript prevents you from using one where the other is expected. This is especially useful for IDs — `UserId` and `PostId` are both strings, but you should never pass a `PostId` where a `UserId` is expected.

### Purposes

- To prevent mixing up structurally identical types (e.g., IDs).
- To enforce domain invariants at compile time.
- To model value objects in Domain-Driven Design.
- To add semantic meaning to primitive types.
- To prevent bugs from passing the wrong string.

### Syntax Rules and Structure

#### Basic Brand

```typescript
type UserId = string & { readonly __brand: 'UserId' };
type PostId = string & { readonly __brand: 'PostId' };

function createUserId(value: string): UserId {
  // Validate the value
  if (!value.startsWith('user_')) {
    throw new Error('Invalid UserId');
  }
  return value as UserId;
}

function createPostId(value: string): PostId {
  if (!value.startsWith('post_')) {
    throw new Error('Invalid PostId');
  }
  return value as PostId;
}

function getUser(id: UserId): void {
  console.log(`Getting user ${id}`);
}

const userId = createUserId('user_123');
const postId = createPostId('post_456');

getUser(userId);   // OK
// getUser(postId); // Error: PostId is not assignable to UserId
// getUser('user_789'); // Error: string is not assignable to UserId
```

#### Generic Brand Helper

```typescript
declare const brand: unique symbol;

type Brand<T, B> = T & { readonly [brand]: B };

type UserId = Brand<string, 'UserId'>;
type PostId = Brand<string, 'PostId'>;
type Email = Brand<string, 'Email'>;
type Cents = Brand<number, 'Cents'>;
```

#### Branded Type with Validation

```typescript
declare const brand: unique symbol;
type Brand<T, B> = T & { readonly [brand]: B };

type Email = Brand<string, 'Email'>;

function parseEmail(value: string): Email {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(value)) {
    throw new Error(`Invalid email: ${value}`);
  }
  return value as Email;
}

function sendEmail(to: Email, subject: string): void {
  console.log(`Sending to ${to}: ${subject}`);
}

const email = parseEmail('alice@example.com');
sendEmail(email, 'Welcome!'); // OK
// sendEmail('bob@example.com', 'Hi'); // Error: string is not assignable to Email
```

#### Branded Types with Template Literals

```typescript
declare const brand: unique symbol;
type Brand<T, B> = T & { readonly [brand]: B };

type HttpMethod = 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';
type Route<T extends string> = Brand<T, 'Route'>;

type ApiRoute = Route<`/api/${string}`>;

function navigate(route: ApiRoute): void {
  console.log(`Navigating to ${route}`);
}

const route = '/api/users' as ApiRoute;
navigate(route); // OK
// navigate('/users' as any); // Not type-safe
```

#### Syntax Rules

- **Use `unique symbol` for the brand** — `declare const brand: unique symbol`.
- **Use `Brand<T, B>` helper** — for consistency.
- **Create values via functions** — with validation.
- **Use `as` assertions sparingly** — only in constructors.
- **Combine with template literals** — for additional constraints.
- **Do not expose the brand** — treat it as phantom.
- **Document branded types** — with JSDoc.
- **Use for IDs, emails, URLs, currency** — semantic types.

#### Constraints and Limitations

- **Brands are compile-time only** — no runtime effect.
- **`as` assertions bypass validation** — use constructor functions.
- **Serialisation loses the brand** — JSON.parse returns plain types.
- **Some libraries do not preserve brands** — ORMs, validation.
- **Overuse adds complexity** — use for critical types.
- **Brands can be hard to debug** — error messages may be verbose.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Branded Types for Domain IDs

```typescript
// branded-types.ts
declare const brand: unique symbol;
type Brand<T, B> = T & { readonly [brand]: B };

// Domain ID types
type UserId = Brand<string, 'UserId'>;
type PostId = Brand<string, 'PostId'>;
type CommentId = Brand<string, 'CommentId'>;

// Value object types
type Email = Brand<string, 'Email'>;
type Cents = Brand<number, 'Cents'>;
type URL = Brand<string, 'URL'>;

// Constructors with validation
function createUserId(value: string): UserId {
  if (!/^user_[a-f0-9]{24}$/.test(value)) {
    throw new Error(`Invalid UserId: ${value}`);
  }
  return value as UserId;
}

function createPostId(value: string): PostId {
  if (!/^post_[a-f0-9]{24}$/.test(value)) {
    throw new Error(`Invalid PostId: ${value}`);
  }
  return value as PostId;
}

function parseEmail(value: string): Email {
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
    throw new Error(`Invalid email: ${value}`);
  }
  return value as Email;
}

function toCents(dollars: number): Cents {
  if (dollars < 0) throw new Error('Amount cannot be negative');
  return Math.round(dollars * 100) as Cents;
}

// Domain functions
interface User {
  id: UserId;
  email: Email;
}

interface Post {
  id: PostId;
  authorId: UserId;
  title: string;
}

function createPost(authorId: UserId, title: string): Post {
  return {
    id: createPostId(`post_${crypto.randomUUID().replace(/-/g, '').slice(0, 24)}`),
    authorId,
    title,
  };
}

function transfer(from: UserId, to: UserId, amount: Cents): void {
  console.log(`Transferring ${amount} cents from ${from} to ${to}`);
}

// Usage
const userId = createUserId('user_abcdef1234567890abcdef12');
const recipientId = createUserId('user_1234567890abcdef12345678');
const email = parseEmail('alice@example.com');
const amount = toCents(49.99);

console.log(userId);   // 'user_abcdef1234567890abcdef12'
console.log(email);    // 'alice@example.com'
console.log(amount);   // 4999

transfer(userId, recipientId, amount); // OK

// These would all be compile errors:
// transfer(recipientId, userId, '4999');     // string is not Cents
// transfer(userId, email, amount);           // Email is not UserId
// const post = createPost(email, 'Hello');   // Email is not UserId
```

**Expected Output:**
```
user_abcdef1234567890abcdef12
alice@example.com
4999
Transferring 4999 cents from user_abcdef1234567890abcdef12 to user_1234567890abcdef12345678
```

**Why this works:** Branded types distinguish `UserId`, `PostId`, `Email`, and `Cents` even though they share underlying primitives. Constructor functions validate at runtime. TypeScript prevents mixing them up at compile time.

### Real-World Cases

- **Domain-Driven Design:** Value objects (`Email`, `Money`, `Address`).
- **Database IDs:** `UserId`, `PostId`, `OrderId`.
- **Financial:** `Cents`, `Dollars`, `Percentage`.
- **Security:** `HashedPassword`, `JwtToken`, `ApiKey`.
- **URLs:** `AbsoluteURL`, `RelativeURL`.

---

## References

- TypeScript Documentation — Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Documentation — Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript Documentation — Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Documentation — Narrowing — https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript Documentation — Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Documentation — Decorators — https://www.typescriptlang.org/docs/handbook/decorators.html
- TypeScript Documentation — Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript Documentation — `infer` — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types
- TypeScript Documentation — `const` Type Parameters — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html#const-type-parameters
- TypeScript Documentation — Variadic Tuple Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-0.html#variadic-tuple-types
- TypeScript Documentation — Standard Decorators — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html#decorators
- TC39 Proposal — Decorators — https://github.com/tc39/proposal-decorators
- NestJS Documentation — Custom Decorators — https://docs.nestjs.com/custom-decorators
- TypeORM Documentation — Decorators — https://typeorm.io/decorators
- class-validator Documentation — https://github.com/typestack/class-validator
- type-fest — Utility Types Collection — https://github.com/sindresorhus/type-fest
- ts-toolbelt — TypeScript Utilities — https://github.com/millsp/ts-toolbelt
- Type Challenges — https://github.com/type-challenges/type-challenges
- Total TypeScript — https://www.totaltypescript.com/
- Matt Pocock — TypeScript Tips — https://www.totaltypescript.com/tips
- TypeScript Deep Dive — https://basarat.gitbook.io/typescript/
- Nominal Typing in TypeScript — https://basarat.gitbook.io/typescript/main-1/nominaltyping
- Branded Types — https://egghead.io/blog/using-branded-types-in-typescript
- tsd — TypeScript Type Testing — https://github.com/SamVerschueren/tsd
- expect-type — Type Testing — https://github.com/mmkal/expect-type