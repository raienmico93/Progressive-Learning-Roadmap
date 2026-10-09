# TypeScript Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** TypeScript is a statically typed superset of JavaScript that adds optional type annotations, interfaces, generics, and compile-time type checking, then compiles to plain JavaScript.

**Technical Definition:** TypeScript is an open-source language developed by Microsoft that extends JavaScript with a static type system. It compiles to standard JavaScript via the `tsc` compiler and is designed for development-time type safety without runtime overhead (types are erased). TypeScript's type system is structural, gradual, and erasure-based — types describe the shape of values rather than their nominal identity, and all type information is removed during compilation.

**Beginner-Friendly Explanation:** Think of TypeScript as JavaScript with a spell-checker for your data. Plain JavaScript lets you put anything anywhere — a number in a variable that's supposed to hold a name, an object without required fields, a function called with the wrong arguments. TypeScript catches these mistakes before you run the code, like a spell-checker catching typos before you send an email. When you're done, TypeScript compiles down to plain JavaScript, so your code runs everywhere JavaScript does.

### Key Characteristics

- **Superset of JavaScript:** All valid JavaScript is valid TypeScript.
- **Structural typing:** Types are compared by shape, not name.
- **Erasure-based:** Types are removed at compile time — no runtime cost.
- **Gradual typing:** You can add types incrementally.
- **Strict mode:** Optional strict flags that enforce stricter type checking.
- **Rich tooling:** Autocomplete, refactoring, and navigation in editors.
- **Powerful type system:** Generics, conditional types, mapped types, template literal types.

### Prerequisites

- **JavaScript fundamentals:** Variables, functions, objects, arrays, classes.
- **ES modules:** `import`/`export` syntax.
- **Node.js or browser environment:** To run compiled JavaScript.
- **Package manager:** npm, pnpm, or Yarn to install TypeScript.
- **Editor support:** VS Code or any editor with TypeScript support.
- **Basic command line:** To run `tsc` and `npm` scripts.

### Related Programming Areas

- **Node.js development:** TypeScript is widely used for backend development.
- **React, Vue, Angular:** Frameworks with first-class TypeScript support.
- **API development:** Type-safe DTOs, validation, and contracts.
- **Testing:** Type-safe test suites with Jest, Vitest, Mocha.
- **Build tooling:** esbuild, swc, ts-node, Vite.
- **Monorepos:** TypeScript project references, shared types.

### Core Concepts

1. **Types** — primitive types, object types, and array typing definitions.
2. **Interfaces** — defining object shapes, extending types, and implementing contracts.
3. **Type Aliases** — creating custom type definitions for structural reusability.
4. **Unions** — allowing values to hold one of several distinct types.
5. **Intersections** — combining multiple structural definitions into a single type.
6. **Generics** — building flexible, reusable components that work with a variety of data types.
7. **Strict Type Checking Flags** — enforcing compiler rules like `strictNullChecks` and `noImplicitAny`.
8. **Enums vs. Const Assertions / Literal Types** — evaluating memory and execution trade-offs.

---

## Core Concept 1: Types (Primitive, Object, Array)

### Definitions

**Core Definition:** Types in TypeScript describe the shape and behaviour of values, from primitive types (string, number, boolean) to complex object and array types.

**Technical Definition:** TypeScript's type system includes primitive types (`string`, `number`, `boolean`, `bigint`, `symbol`, `null`, `undefined`), object types (`object`, interfaces, classes), array types (`T[]`, `Array<T>`), tuple types (`[T, U]`), function types (`(a: T) => U`), and special types (`any`, `unknown`, `never`, `void`). Type annotations are written with a colon after the variable or parameter name (`let x: number = 5`). Types are inferred when not explicitly annotated, and are erased at compile time.

**Beginner-Friendly Explanation:** Types are labels you put on your data. A `string` label means "this holds text," a `number` label means "this holds a number," and an `object` label means "this has specific fields." TypeScript checks that you never put the wrong type of data in the wrong place.

### Purposes

- To document the expected shape and behaviour of values.
- To catch type errors at compile time rather than runtime.
- To enable editor autocomplete and refactoring.
- To serve as executable documentation.
- To enable safe refactoring across large codebases.

### Syntax Rules and Structure

#### Primitive Types

```typescript
let name: string = 'Alice';
let age: number = 30;
let isActive: boolean = true;
let big: bigint = 100n;
let id: symbol = Symbol('id');
let nothing: null = null;
let missing: undefined = undefined;
```

| Type | Description | Example |
|------|-------------|---------|
| `string` | Text | `'hello'`, `"world"` |
| `number` | Integer or float | `42`, `3.14` |
| `boolean` | True/false | `true`, `false` |
| `bigint` | Arbitrary-precision integer | `100n` |
| `symbol` | Unique identifier | `Symbol('id')` |
| `null` | Intentional absence | `null` |
| `undefined` | Unintentional absence | `undefined` |

#### Special Types

| Type | Description | Use Case |
|------|-------------|----------|
| `any` | Disables type checking | Escape hatch (avoid) |
| `unknown` | Safe counterpart to `any` | Untrusted input |
| `never` | No possible value | Exhaustive checks |
| `void` | No return value | Functions with side effects |
| `object` | Any non-primitive | Generic object references |

#### Object Types

```typescript
// Inline object type
let user: { name: string; age: number } = { name: 'Alice', age: 30 };

// Optional properties
let config: { host: string; port?: number } = { host: 'localhost' };

// Readonly properties
let point: { readonly x: number; readonly y: number } = { x: 1, y: 2 };
// point.x = 5; // Error: Cannot assign to 'x' because it is a read-only property

// Index signatures
let scores: { [key: string]: number } = { alice: 95, bob: 87 };
```

#### Array Types

```typescript
// Array of strings
let names: string[] = ['Alice', 'Bob'];
let names2: Array<string> = ['Alice', 'Bob'];

// Array of objects
let users: { name: string; age: number }[] = [
  { name: 'Alice', age: 30 },
  { name: 'Bob', age: 25 },
];

// Tuples — fixed-length arrays with known types
let pair: [string, number] = ['Alice', 30];
let rgb: [number, number, number] = [255, 128, 0];

// Readonly arrays
let readonlyNames: readonly string[] = ['Alice', 'Bob'];
// readonlyNames.push('Charlie'); // Error

// Nested arrays
let matrix: number[][] = [[1, 2], [3, 4]];
```

#### Function Types

```typescript
// Function with typed parameters and return
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => a * b;

// Function type annotation
let operation: (a: number, b: number) => number;
operation = add;

// Optional and default parameters
function greet(name: string, greeting: string = 'Hello'): string {
  return `${greeting}, ${name}!`;
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((a, b) => a + b, 0);
}
```

#### Syntax Rules

- **Use `: type` after the variable or parameter name.**
- **Let TypeScript infer types** when obvious — avoid redundant annotations.
- **Use `readonly`** for properties that should not change.
- **Use `?`** for optional properties and parameters.
- **Use `[]` or `Array<T>`** for arrays — pick one style and be consistent.
- **Use tuples** for fixed-length arrays.
- **Avoid `any`** — use `unknown` for untrusted input.
- **Enable strict mode** — `"strict": true` in `tsconfig.json`.

#### Constraints and Limitations

- **Types are erased at runtime** — no runtime type checking.
- **Structural typing can surprise** — objects with the same shape are compatible.
- **`any` disables safety** — use sparingly.
- **Type annotations add verbosity** — balance with inference.
- **No runtime validation** — use Zod, Joi, or class-validator for that.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Types in Practice

```typescript
// primitives.ts
// Primitive types
let username: string = 'alice';
let age: number = 30;
let isActive: boolean = true;

// Type inference — TypeScript infers the type
let inferredName = 'Bob'; // inferred as string
let inferredAge = 25;     // inferred as number

// Object types
interface User {
  id: string;
  name: string;
  email: string;
  age?: number; // Optional
  readonly createdAt: Date; // Readonly
}

const user: User = {
  id: 'user-1',
  name: 'Alice',
  email: 'alice@example.com',
  createdAt: new Date(),
};

// user.createdAt = new Date(); // Error: readonly

// Array types
const users: User[] = [user];
const names: Array<string> = users.map((u) => u.name);

// Tuple types
const coordinates: [number, number] = [10.5, 20.3];
const [lat, lng] = coordinates;

// Function types
function formatUser(user: User): string {
  return `${user.name} (${user.email})`;
}

const formatUserArrow = (user: User): string => `${user.name} (${user.email})`;

// Function type annotation
type Formatter = (user: User) => string;
const formatter: Formatter = formatUser;

console.log(formatUser(user)); // 'Alice (alice@example.com)'
```

**Expected Output:**
```
Alice (alice@example.com)
```

**Why this works:** Types describe the shape of data. TypeScript checks that `user` has all required fields, that `users` is an array of `User`, and that `formatUser` is called with a `User`. Errors are caught at compile time.

### Real-World Cases

- **API responses:** Typing the shape of JSON responses from APIs.
- **Configuration objects:** Typing config files to catch missing fields.
- **Function parameters:** Ensuring functions are called with the right arguments.
- **React props:** Typing component props.

---

## Core Concept 2: Interfaces

### Definitions

**Core Definition:** An interface declares the shape of an object — its properties, methods, and their types — serving as a contract that objects must satisfy.

**Technical Definition:** Interfaces in TypeScript define the structure of objects without providing an implementation. They can be extended (inheritance), implemented by classes, and merged (declaration merging). Interfaces support optional properties (`?`), readonly properties (`readonly`), index signatures (`[key: string]: T`), method signatures, and call signatures. Unlike type aliases, interfaces can be extended and merged, making them ideal for public APIs and library definitions.

**Beginner-Friendly Explanation:** An interface is like a job description. It says "anyone applying for this role must have these skills and experience." If an object doesn't match the interface, TypeScript rejects it. Interfaces are about the shape of data, not the identity of the class.

### Purposes

- To define contracts for objects and classes.
- To document the shape of data structures.
- To enable structural typing and duck typing.
- To support extension and implementation.
- To provide a stable public API for libraries.

### Syntax Rules and Structure

#### Basic Interface

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age?: number; // Optional
  readonly createdAt: Date; // Readonly
}
```

#### Methods

```typescript
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
  multiply?: (a: number, b: number) => number; // Optional method
}
```

#### Index Signatures

```typescript
interface StringMap {
  [key: string]: string;
}

const map: StringMap = { a: '1', b: '2' };
```

#### Extending Interfaces

```typescript
interface Animal {
  name: string;
  age: number;
}

interface Dog extends Animal {
  breed: string;
  bark(): void;
}

interface Cat extends Animal {
  indoor: boolean;
  meow(): void;
}

// Multiple extension
interface Pet extends Animal, Serializable {
  owner: string;
}

// Extension with override
interface Employee extends Person {
  age: number | null; // Override with compatible type
}
```

#### Implementing Interfaces

```typescript
interface Shape {
  area(): number;
  perimeter(): number;
}

class Circle implements Shape {
  constructor(private radius: number) {}

  area(): number {
    return Math.PI * this.radius ** 2;
  }

  perimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}

  area(): number {
    return this.width * this.height;
  }

  perimeter(): number {
    return 2 * (this.width + this.height);
  }
}
```

#### Declaration Merging

```typescript
interface Window {
  myCustomProperty: string;
}

interface Window {
  anotherProperty: number;
}

// Both declarations merge — Window now has both properties
window.myCustomProperty = 'hello';
window.anotherProperty = 42;
```

#### Syntax Rules

- **Use `interface` for object shapes** — especially public APIs.
- **Use `extends` for inheritance** — a single interface can extend multiple.
- **Use `implements` for classes** — a class must provide all members.
- **Use `?` for optional properties.**
- **Use `readonly` for immutable properties.**
- **Use index signatures** for dynamic keys.
- **Use declaration merging** for augmenting third-party types.
- **Prefer interfaces over type aliases** for object shapes (for merging and performance).

#### Constraints and Limitations

- **Interfaces cannot express unions** — use type aliases for unions.
- **Interfaces cannot express primitives** — `type ID = string` is a type alias.
- **Declaration merging can be surprising** — global types can be augmented.
- **Structural typing** — an object with the same shape satisfies the interface.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Interfaces and Implementation

```typescript
// interfaces.ts
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

interface User {
  id: string;
  name: string;
  email: string;
}

class InMemoryUserRepository implements Repository<User> {
  private users = new Map<string, User>();

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) ?? null;
  }

  async findAll(): Promise<User[]> {
    return [...this.users.values()];
  }

  async save(user: User): Promise<User> {
    this.users.set(user.id, user);
    return user;
  }

  async delete(id: string): Promise<void> {
    this.users.delete(id);
  }
}

// Usage
(async () => {
  const repo = new InMemoryUserRepository();
  await repo.save({ id: '1', name: 'Alice', email: 'alice@example.com' });
  const user = await repo.findById('1');
  console.log(user); // { id: '1', name: 'Alice', email: 'alice@example.com' }
})();
```

**Expected Output:**
```
{ id: '1', name: 'Alice', email: 'alice@example.com' }
```

**Why this works:** The `Repository<T>` interface defines the contract. `InMemoryUserRepository` implements it. TypeScript checks that all methods are present and correctly typed. The `User` interface defines the entity shape.

### Real-World Cases

- **API contracts:** Defining request and response shapes.
- **Repositories:** Defining persistence contracts.
- **React props:** Typing component props.
- **Library APIs:** Providing stable public interfaces.

---

## Core Concept 3: Type Aliases

### Definitions

**Core Definition:** A type alias creates a new name for an existing type, enabling reusable, readable type definitions for primitives, unions, intersections, tuples, and object shapes.

**Technical Definition:** Type aliases (`type Alias = Type`) assign a name to any type expression. Unlike interfaces, type aliases can represent primitive types, unions, intersections, tuples, and mapped types. They cannot be extended or merged, but they can be composed via intersections. Type aliases are preferred for unions, primitives, and complex type expressions; interfaces are preferred for object shapes that need extension or merging.

**Beginner-Friendly Explanation:** A type alias is like a nickname for a type. Instead of writing `string | number | boolean` everywhere, you write `type Scalar = string | number | boolean`. It's a shorthand that makes your code more readable and easier to maintain.

### Purposes

- To give meaningful names to complex types.
- To define unions, intersections, and tuples.
- To create reusable type definitions.
- To compose types from other types.
- To improve code readability and maintainability.

### Syntax Rules and Structure

#### Basic Type Alias

```typescript
type ID = string;
type Age = number;
type Callback = () => void;
type AsyncCallback<T> = () => Promise<T>;
```

#### Union Type Alias

```typescript
type Status = 'pending' | 'active' | 'inactive' | 'deleted';
type Result<T> = { success: true; data: T } | { success: false; error: string };
```

#### Intersection Type Alias

```typescript
type Timestamped = {
  createdAt: Date;
  updatedAt: Date;
};

type User = {
  id: string;
  name: string;
} & Timestamped;
```

#### Tuple Type Alias

```typescript
type Point = [number, number];
type RGB = [red: number, green: number, blue: number];
type Response<T> = [status: number, data: T];
```

#### Function Type Alias

```typescript
type Comparator<T> = (a: T, b: T) => number;
type Predicate<T> = (value: T) => boolean;
type Mapper<T, U> = (value: T) => U;
```

#### Conditional Type Alias

```typescript
type IsString<T> = T extends string ? true : false;
type NonNullable<T> = T extends null | undefined ? never : T;
type Awaited<T> = T extends Promise<infer U> ? U : T;
```

#### Mapped Type Alias

```typescript
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

type Partial<T> = {
  [K in keyof T]?: T[K];
};

type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

#### Syntax Rules

- **Use `type Alias = ...`** to create an alias.
- **Use type aliases for unions, intersections, tuples, and primitives.**
- **Use interfaces for object shapes that need extension or merging.**
- **Use generic type aliases** for reusable components.
- **Use conditional and mapped types** for advanced type transformations.
- **Prefer type aliases for readability** — give complex types meaningful names.
- **Compose type aliases** with intersections.
- **Use `infer`** in conditional types to extract types.

#### Constraints and Limitations

- **Type aliases cannot be extended** — use intersections instead.
- **Type aliases cannot be merged** — unlike interfaces.
- **Recursive type aliases** are supported but must be carefully constructed.
- **Some type aliases cannot be expressed as interfaces** — unions, tuples, primitives.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Type Aliases in Practice

```typescript
// type-aliases.ts
// Primitive alias
type UserID = string;
type Cents = number;

// Union alias
type Status = 'pending' | 'active' | 'inactive' | 'deleted';
type Result<T> = { success: true; data: T } | { success: false; error: string };

// Intersection alias
type Timestamped = {
  createdAt: Date;
  updatedAt: Date;
};

type BaseEntity = {
  id: UserID;
} & Timestamped;

// Object alias
type User = BaseEntity & {
  name: string;
  email: string;
  status: Status;
};

// Tuple alias
type Point = [number, number];
type HttpResponse<T> = [status: number, data: T];

// Function alias
type Comparator<T> = (a: T, b: T) => number;
type Predicate<T> = (value: T) => boolean;

// Conditional alias
type NonNullable<T> = T extends null | undefined ? never : T;
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

// Usage
const user: User = {
  id: 'user-1',
  name: 'Alice',
  email: 'alice@example.com',
  status: 'active',
  createdAt: new Date(),
  updatedAt: new Date(),
};

function processResult<T>(result: Result<T>): T {
  if (result.success) {
    return result.data;
  }
  throw new Error(result.error);
}

const result: Result<number> = { success: true, data: 42 };
console.log(processResult(result)); // 42
```

**Expected Output:**
```
42
```

**Why this works:** Type aliases give meaningful names to complex types. The `Result<T>` alias models a success/failure outcome. The `User` alias composes multiple types. TypeScript checks that all properties are present and correctly typed.

### Real-World Cases

- **API responses:** `type ApiResponse<T> = { data: T; error?: string }`.
- **State machines:** `type Status = 'idle' | 'loading' | 'success' | 'error'`.
- **Utility types:** `type Nullable<T> = T | null`.
- **Domain types:** `type Email = string & { __brand: 'Email' }`.

---

## Core Concept 4: Unions

### Definitions

**Core Definition:** A union type allows a value to be one of several distinct types, expressed with the `|` operator.

**Technical Definition:** Union types (`A | B`) represent a value that can be either type A or type B. TypeScript requires narrowing (via `typeof`, `instanceof`, `in`, discriminant properties, or user-defined type guards) before accessing type-specific members. Unions are the foundation of TypeScript's discriminated union pattern, which models algebraic data types and enables exhaustive checking via `never`.

**Beginner-Friendly Explanation:** A union is like a variable that can hold one of several types. For example, a `string | number` variable can hold either a string or a number, but not both at once. Before you use it, TypeScript requires you to check which type it is — like checking whether the box contains a book or a DVD before deciding what to do with it.

### Purposes

- To model values that can be one of several types.
- To enable discriminated unions for state machines and result types.
- To support nullable types (`T | null`).
- To enable exhaustive type checking via `never`.
- To model API responses with different shapes.

### Syntax Rules and Structure

#### Basic Union

```typescript
let id: string | number;
id = 'abc';  // OK
id = 123;    // OK
// id = true; // Error
```

#### Union with Literals

```typescript
type Status = 'pending' | 'active' | 'inactive';
type Direction = 'north' | 'south' | 'east' | 'west';
type Digit = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9;
```

#### Nullable Types

```typescript
type Nullable<T> = T | null;
type Optional<T> = T | undefined;
type Maybe<T> = T | null | undefined;

let user: User | null = null;
```

#### Discriminated Unions

```typescript
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

function handle<T>(result: Result<T>): T {
  if (result.success) {
    return result.data; // Narrowed to success branch
  }
  throw new Error(result.error); // Narrowed to failure branch
}
```

#### Exhaustive Checking

```typescript
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number }
  | { kind: 'rectangle'; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.side ** 2;
    case 'rectangle':
      return shape.width * shape.height;
    default:
      const _exhaustive: never = shape;
      throw new Error(`Unhandled shape: ${_exhaustive}`);
  }
}
```

#### Type Narrowing

```typescript
function format(value: string | number | boolean): string {
  if (typeof value === 'string') {
    return value.toUpperCase(); // value is string
  }
  if (typeof value === 'number') {
    return value.toFixed(2); // value is number
  }
  return value ? 'yes' : 'no'; // value is boolean
}
```

#### Syntax Rules

- **Use `|` to combine types** — `A | B | C`.
- **Use `typeof` to narrow primitives.**
- **Use `instanceof` to narrow classes.**
- **Use `in` to narrow by property presence.**
- **Use discriminated unions** for state machines and results.
- **Use `never` for exhaustive checks.**
- **Use `null` and `undefined` unions** for optional values.
- **Enable `strictNullChecks`** — required for safe null handling.

#### Constraints and Limitations

- **Narrowing is required** — you cannot access type-specific members without it.
- **Complex unions are hard to read** — use discriminated unions.
- **Union of object types** — only common properties are accessible without narrowing.
- **Type guards must be correct** — user-defined guards can lie.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Discriminated Unions for State Machines

```typescript
// state-machine.ts
type LoadingState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: string[] }
  | { status: 'error'; message: string };

function renderState(state: LoadingState): string {
  switch (state.status) {
    case 'idle':
      return 'Ready to load.';
    case 'loading':
      return 'Loading...';
    case 'success':
      return `Loaded ${state.data.length} items.`;
    case 'error':
      return `Error: ${state.message}`;
    default:
      const _exhaustive: never = state;
      throw new Error(`Unhandled state: ${JSON.stringify(_exhaustive)}`);
  }
}

console.log(renderState({ status: 'idle' }));                        // 'Ready to load.'
console.log(renderState({ status: 'loading' }));                     // 'Loading...'
console.log(renderState({ status: 'success', data: ['a', 'b'] }));   // 'Loaded 2 items.'
console.log(renderState({ status: 'error', message: 'Failed' }));    // 'Error: Failed'
```

**Expected Output:**
```
Ready to load.
Loading...
Loaded 2 items.
Error: Failed
```

**Why this works:** The discriminated union has a `status` field that TypeScript uses to narrow the type in each `case`. The `never` check ensures all cases are handled. If a new status is added, TypeScript reports a compile error.

### Real-World Cases

- **State machines:** UI states (idle, loading, success, error).
- **API results:** Success/failure responses.
- **Form validation:** Valid/invalid states with error messages.
- **Redux actions:** Discriminated action types.

---

## Core Concept 5: Intersections

### Definitions

**Core Definition:** An intersection type combines multiple types into one, requiring a value to satisfy all the combined types simultaneously.

**Technical Definition:** Intersection types (`A & B`) represent a value that has all properties of A and all properties of B. Intersections are used for composition — combining multiple interfaces or type aliases into a single type. Unlike unions (OR), intersections are AND. Intersections are commonly used for mixins, decorators, and composing utility types.

**Beginner-Friendly Explanation:** An intersection is like a venn diagram where both circles must overlap. If a value is `A & B`, it must have all the properties of A and all the properties of B. It's the "AND" of types, whereas a union is the "OR."

### Purposes

- To compose multiple types into one.
- To implement mixins and decorators.
- To combine utility types.
- To add properties to existing types.
- To model complex domain objects.

### Syntax Rules and Structure

#### Basic Intersection

```typescript
type Person = { name: string };
type Employee = { employeeId: string };

type StaffMember = Person & Employee;
// StaffMember has both name and employeeId
```

#### Composing Interfaces

```typescript
interface Serializable {
  serialize(): string;
}

interface Loggable {
  log(): void;
}

interface Entity extends Serializable, Loggable {
  id: string;
}

// Equivalent to:
type Entity = Serializable & Loggable & { id: string };
```

#### Mixins

```typescript
type Constructor<T = {}> = new (...args: any[]) => T;

function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    createdAt = new Date();
    updatedAt = new Date();
  };
}

function SoftDeletable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    deletedAt: Date | null = null;
    softDelete() {
      this.deletedAt = new Date();
    }
  };
}

class User {
  constructor(public name: string) {}
}

const TimestampedUser = Timestamped(SoftDeletable(User));
const user = new TimestampedUser('Alice');
console.log(user.createdAt); // Date
user.softDelete();
console.log(user.deletedAt); // Date
```

#### Syntax Rules

- **Use `&` to combine types** — `A & B`.
- **Use intersections for composition** — combining multiple shapes.
- **Use intersections with utility types** — `Partial<T> & { required: string }`.
- **Use intersections for mixins** — combining class behaviours.
- **Use interfaces with `extends`** for simpler composition.
- **Avoid conflicting property types** — `{ x: string } & { x: number }` is `never`.

#### Constraints and Limitations

- **Conflicting properties** — intersections with incompatible properties produce `never`.
- **Harder to read** — complex intersections can be confusing.
- **Interfaces are preferred** for simple extension.
- **Intersections cannot express unions** — use type aliases.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Composing Types with Intersections

```typescript
// intersections.ts
interface Identifiable {
  id: string;
}

interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

interface SoftDeletable {
  deletedAt: Date | null;
}

// Compose a full entity type
type Entity = Identifiable & Timestamped & SoftDeletable;

interface User extends Entity {
  name: string;
  email: string;
}

// Utility intersection
type WithRequired<T, K extends keyof T> = T & { [P in K]-?: T[P] };

type PartialUser = Partial<User>;
type UserWithEmail = WithRequired<PartialUser, 'email'>;

// Usage
const user: User = {
  id: 'user-1',
  name: 'Alice',
  email: 'alice@example.com',
  createdAt: new Date(),
  updatedAt: new Date(),
  deletedAt: null,
};

console.log(user.id); // 'user-1'
console.log(user.deletedAt); // null

// Mixin example
type Constructor<T = {}> = new (...args: any[]) => T;

function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActive = false;
    activate() { this.isActive = true; }
    deactivate() { this.isActive = false; }
  };
}

class BaseUser {
  constructor(public name: string) {}
}

const ActiveUser = Activatable(BaseUser);
const activeUser = new ActiveUser('Bob');
activeUser.activate();
console.log(activeUser.isActive); // true
```

**Expected Output:**
```
user-1
null
true
```

**Why this works:** The `Entity` intersection composes three interfaces. The `User` interface extends `Entity`. The mixin `Activatable` adds behaviour to `BaseUser`. TypeScript verifies that all properties are present and correctly typed.

### Real-World Cases

- **Domain entities:** Composing `Identifiable`, `Timestamped`, and `SoftDeletable`.
- **Mixins:** Adding behaviour to classes without inheritance.
- **Utility types:** Composing `Partial`, `Required`, `Pick`, `Omit`.
- **React props:** Combining base props with component-specific props.

---

## Core Concept 6: Generics

### Definitions

**Core Definition:** Generics allow types to be parameterised, enabling reusable components that work with a variety of data types while preserving type safety.

**Technical Definition:** Generics introduce type parameters (`<T>`) that are substituted with concrete types at the call site. They enable type-safe collections, functions, and classes without sacrificing flexibility. Generic constraints (`<T extends U>`) restrict the types that can be used. Default type parameters (`<T = string>`) provide fallbacks. Generics are the foundation of TypeScript's utility types (`Array<T>`, `Promise<T>`, `Partial<T>`, `Record<K, V>`).

**Beginner-Friendly Explanation:** A generic is like a template that works with any type. Instead of writing a separate function for `string[]`, `number[]`, and `User[]`, you write one generic function that works with all of them — and TypeScript remembers which type you passed in. It's like a cookie cutter that can make cookies in any shape you want, but always produces the exact shape you asked for.

### Purposes

- To create reusable components that work with any type.
- To preserve type information through transformations.
- To avoid code duplication while maintaining type safety.
- To build type-safe collections and data structures.
- To create utility types and higher-order functions.

### Syntax Rules and Structure

#### Generic Function

```typescript
function identity<T>(value: T): T {
  return value;
}

const str = identity('hello');  // T = string
const num = identity(42);       // T = number
const obj = identity({ a: 1 }); // T = { a: 1 }
```

#### Generic Interface

```typescript
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}
```

#### Generic Class

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  get size(): number {
    return this.items.length;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop()); // 2
```

#### Generic Constraints

```typescript
// T must have a length property
function logLength<T extends { length: number }>(item: T): void {
  console.log(item.length);
}

logLength('hello');        // OK
logLength([1, 2, 3]);      // OK
logLength({ length: 10 }); // OK
// logLength(42);          // Error: number has no length
```

#### Multiple Type Parameters

```typescript
function pair<K, V>(key: K, value: V): [K, V] {
  return [key, value];
}

const p = pair('name', 'Alice'); // [string, string]
const p2 = pair(1, { name: 'Alice' }); // [number, { name: string }]
```

#### Default Type Parameters

```typescript
interface ApiResponse<T = unknown> {
  data: T;
  status: number;
}

const response: ApiResponse = { data: 'anything', status: 200 };
const typedResponse: ApiResponse<User> = { data: user, status: 200 };
```

#### Generic Utility Types

```typescript
type Partial<T> = { [K in keyof T]?: T[K] };
type Required<T> = { [K in keyof T]-?: T[K] };
type Readonly<T> = { readonly [K in keyof T]: T[K] };
type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
type Record<K extends keyof any, T> = { [P in K]: T };
```

#### Syntax Rules

- **Use `<T>` for type parameters** — `T`, `U`, `V`, `K`, `V`.
- **Use `extends` for constraints** — `<T extends U>`.
- **Use `=` for defaults** — `<T = string>`.
- **Use multiple parameters** — `<K, V>`.
- **Use utility types** — `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`.
- **Use `keyof` for property keys** — `<T, K extends keyof T>`.
- **Use `infer` in conditional types** — `T extends Promise<infer U> ? U : T`.

#### Constraints and Limitations

- **Generics add complexity** — use only when needed.
- **Type inference may fail** — explicit annotations may be required.
- **Generic constraints must be satisfiable** — `<T extends string>` excludes numbers.
- **Higher-kinded types are not supported** — TypeScript lacks direct HKT support.
- **Runtime type information is not available** — generics are erased.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Generic Repository and Utility Functions

```typescript
// generics.ts
// Generic repository interface
interface Entity {
  id: string;
}

interface Repository<T extends Entity> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

// Generic in-memory implementation
class InMemoryRepository<T extends Entity> implements Repository<T> {
  protected items = new Map<string, T>();

  async findById(id: string): Promise<T | null> {
    return this.items.get(id) ?? null;
  }

  async findAll(): Promise<T[]> {
    return [...this.items.values()];
  }

  async save(entity: T): Promise<T> {
    this.items.set(entity.id, entity);
    return entity;
  }

  async delete(id: string): Promise<void> {
    this.items.delete(id);
  }
}

// Domain entity
interface User extends Entity {
  name: string;
  email: string;
}

// Generic utility functions
function groupBy<T, K extends keyof T>(items: T[], key: K): Map<T[K], T[]> {
  const map = new Map<T[K], T[]>();
  for (const item of items) {
    const group = map.get(item[key]) ?? [];
    group.push(item);
    map.set(item[key], group);
  }
  return map;
}

function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((item) => item[key]);
}

// Usage
(async () => {
  const userRepo = new InMemoryRepository<User>();
  await userRepo.save({ id: '1', name: 'Alice', email: 'alice@example.com' });
  await userRepo.save({ id: '2', name: 'Bob', email: 'bob@example.com' });

  const users = await userRepo.findAll();
  console.log('Users:', users.length);

  const emails = pluck(users, 'email');
  console.log('Emails:', emails);

  const grouped = groupBy(users, 'name');
  console.log('Grouped size:', grouped.size);
})();
```

**Expected Output:**
```
Users: 2
Emails: [ 'alice@example.com', 'bob@example.com' ]
Grouped size: 2
```

**Why this works:** The `Repository<T extends Entity>` interface is generic over any entity type. `InMemoryRepository<T>` implements it. `groupBy` and `pluck` are generic over `T` and `K`, preserving type information. TypeScript ensures that `pluck(users, 'email')` returns `string[]`.

### Real-World Cases

- **Repositories:** Generic CRUD interfaces for any entity.
- **API clients:** Generic HTTP clients with typed responses.
- **Utility functions:** `map`, `filter`, `reduce`, `groupBy`, `pluck`.
- **Data structures:** Generic stacks, queues, trees, graphs.
- **React components:** Generic components and hooks.

---

## Core Concept 7: Strict Type Checking Flags

### Definitions

**Core Definition:** Strict type checking flags are compiler options that enable stricter type checking, catching more errors at compile time.

**Technical Definition:** The `strict` flag in `tsconfig.json` enables a family of strict checks: `strictNullChecks` (null and undefined are not assignable to other types), `noImplicitAny` (disallow implicit `any`), `strictFunctionTypes` (contravariant function parameter checking), `strictBindCallApply` (strict typing for `bind`, `call`, `apply`), `strictPropertyInitialization` (class properties must be initialised), `noImplicitThis` (disallow implicit `this`), `alwaysStrict` (emit `"use strict"`). Additional flags include `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noImplicitReturns`, `noFallthroughCasesInSwitch`, and `noUnusedLocals`.

**Beginner-Friendly Explanation:** Strict flags are like the difference between a lenient teacher and a strict teacher. The lenient teacher lets small mistakes slide. The strict teacher catches every mistake and requires you to fix it. Strict mode catches more bugs at compile time, making your code safer — but it also requires more explicit typing.

### Purposes

- To catch more errors at compile time.
- To prevent null/undefined bugs.
- To eliminate implicit `any` types.
- To enforce consistent type safety across the codebase.
- To improve code quality and maintainability.
- To align with best practices.

### Syntax Rules and Structure

#### Recommended tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],

    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noPropertyAccessFromIndexSignature": true,
    "allowUnreachableCode": false,
    "allowUnusedLabels": false,

    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true,

    "outDir": "./dist",
    "rootDir": "./src",
    "sourceMap": true,
    "declaration": true,
    "declarationMap": true,

    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

#### Strict Flags Explained

| Flag | What It Enforces |
|------|------------------|
| `strict` | Enables all strict flags below |
| `strictNullChecks` | `null` and `undefined` are not assignable |
| `noImplicitAny` | No implicit `any` types |
| `strictFunctionTypes` | Contravariant parameter checking |
| `strictBindCallApply` | Strict typing for `bind`, `call`, `apply` |
| `strictPropertyInitialization` | Class properties must be initialised |
| `noImplicitThis` | No implicit `this` |
| `alwaysStrict` | Emit `"use strict"` |
| `noUncheckedIndexedAccess` | Index access returns `T \| undefined` |
| `exactOptionalPropertyTypes` | Distinguishes `undefined` from missing |
| `noImplicitOverride` | Requires `override` keyword |
| `noImplicitReturns` | All code paths must return |
| `noFallthroughCasesInSwitch` | No fallthrough in switch |
| `noUnusedLocals` | No unused local variables |
| `noUnusedParameters` | No unused parameters |
| `noPropertyAccessFromIndexSignature` | Requires bracket notation for index signatures |

#### Syntax Rules

- **Always enable `strict: true`** — it is the baseline.
- **Enable `noUncheckedIndexedAccess`** — catches array/object index bugs.
- **Enable `exactOptionalPropertyTypes`** — distinguishes `undefined` from missing.
- **Enable `noImplicitOverride`** — requires explicit `override`.
- **Enable `noImplicitReturns`** — catches missing return paths.
- **Enable `noFallthroughCasesInSwitch`** — catches fallthrough bugs.
- **Enable `noUnusedLocals` and `noUnusedParameters`** — catches dead code.
- **Use `skipLibCheck: true`** — for faster compilation.
- **Use `esModuleInterop: true`** — for CommonJS compatibility.
- **Use `isolatedModules: true`** — for compatibility with esbuild/swc.

#### Constraints and Limitations

- **Stricter flags may break existing code** — gradual adoption may be needed.
- **More explicit typing required** — more verbose code.
- **`noUncheckedIndexedAccess` can be noisy** — many `undefined` checks.
- **`exactOptionalPropertyTypes` may break libraries** — check compatibility.
- **`skipLibCheck` hides errors in dependencies** — trade-off between speed and safety.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Strict Mode in Practice

```typescript
// strict-mode.ts
// With strictNullChecks, null and undefined are not assignable
function greet(name: string | null): string {
  // Error without narrowing: name is possibly null
  if (name === null) {
    return 'Hello, stranger!';
  }
  return `Hello, ${name}!`;
}

// With noImplicitAny, parameters must be typed
function process(data: unknown): void {
  if (typeof data === 'string') {
    console.log(data.toUpperCase());
  } else if (typeof data === 'number') {
    console.log(data.toFixed(2));
  } else {
    console.log('Unknown type');
  }
}

// With strictPropertyInitialization, class properties must be initialised
class User {
  name: string;
  email: string;

  constructor(name: string, email: string) {
    this.name = name;
    this.email = email;
  }
}

// With noUncheckedIndexedAccess, index access returns T | undefined
const users: User[] = [new User('Alice', 'alice@example.com')];
const firstUser = users[0];
if (firstUser) {
  console.log(firstUser.name); // 'Alice'
}

// With exactOptionalPropertyTypes, undefined is distinct from missing
interface Config {
  host: string;
  port?: number; // Can be missing, but not explicitly undefined
}

const config1: Config = { host: 'localhost' };           // OK
const config2: Config = { host: 'localhost', port: 3000 }; // OK
// const config3: Config = { host: 'localhost', port: undefined }; // Error

// With noImplicitReturns, all code paths must return
function classify(value: number): 'positive' | 'negative' | 'zero' {
  if (value > 0) return 'positive';
  if (value < 0) return 'negative';
  return 'zero';
}

// With noFallthroughCasesInSwitch, fallthrough is an error
function describe(value: string): string {
  switch (value) {
    case 'a':
      return 'Letter A';
    case 'b':
      return 'Letter B';
    default:
      return 'Other';
  }
}

console.log(greet('Alice'));       // 'Hello, Alice!'
console.log(greet(null));          // 'Hello, stranger!'
process('hello');                   // 'HELLO'
process(3.14159);                   // '3.14'
console.log(classify(5));           // 'positive'
console.log(describe('a'));         // 'Letter A'
```

**Expected Output:**
```
Hello, Alice!
Hello, stranger!
HELLO
3.14
positive
Letter A
```

**Why this works:** Strict flags catch null/undefined bugs, implicit `any`, uninitialised properties, unchecked index access, and missing return paths. The code must explicitly handle all cases, resulting in safer code.

### Real-World Cases

- **New projects:** Always enable `strict: true`.
- **Legacy migration:** Gradually enable flags, starting with `strictNullChecks`.
- **Libraries:** Publish with strict types for consumers.
- **Monorepos:** Enforce strict mode across all packages.

---

## Core Concept 8: Enums vs. Const Assertions / Literal Types

### Definitions

**Core Definition:** Enums create named constants with a dedicated type, while `as const` assertions and literal types create type-safe constant objects without runtime overhead.

**Technical Definition:** TypeScript `enum` declares a set of named constants. Numeric enums are compiled to a reverse-mapped object (`{ 0: 'A', A: 0 }`), string enums to a simple object (`{ A: 'A' }`), and `const enum` is inlined at compile time. `as const` assertions create deeply readonly literal types — the object is preserved at runtime, but its type is narrowed to literal types. The trade-offs: enums add runtime code (especially numeric enums with reverse mapping), while `as const` is zero-cost but lacks the `enum` keyword. Modern TypeScript style increasingly prefers `as const` for its tree-shaking friendliness and compatibility with `isolatedModules`.

**Beginner-Friendly Explanation:** An enum is like a labelled set of options — `Color.Red`, `Color.Green`. A `const` assertion is like writing the options directly as an object and telling TypeScript "these values will never change." Both work, but enums generate extra JavaScript code, while `as const` is zero-cost. For modern projects, `as const` is often preferred because it produces less code and works better with tree-shaking.

### Purposes

- To define named constants with type safety.
- To model finite sets of values.
- To improve code readability.
- To avoid magic strings and numbers.
- To enable exhaustive checking with discriminated unions.

### Syntax Rules and Structure

#### Numeric Enum

```typescript
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right, // 3
}

// Custom values
enum StatusCode {
  OK = 200,
  NotFound = 404,
  ServerError = 500,
}
```

**Compiled output:**
```javascript
var Direction;
(function (Direction) {
  Direction[Direction["Up"] = 0] = "Up";
  Direction[Direction["Down"] = 1] = "Down";
  // ...
})(Direction || (Direction = {}));
```

#### String Enum

```typescript
enum Status {
  Pending = 'pending',
  Active = 'active',
  Inactive = 'inactive',
}

// Compiled output (no reverse mapping):
var Status;
(function (Status) {
  Status["Pending"] = "pending";
  Status["Active"] = "active";
  Status["Inactive"] = "inactive";
})(Status || (Status = {}));
```

#### Const Enum

```typescript
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}

const dir = Direction.Up; // Inlined to 0 at compile time
```

#### Const Assertion (Modern Alternative)

```typescript
const Direction = {
  Up: 'up',
  Down: 'down',
  Left: 'left',
  Right: 'right',
} as const;

type Direction = typeof Direction[keyof typeof Direction];
// type Direction = 'up' | 'down' | 'left' | 'right'

const dir: Direction = Direction.Up; // 'up'
```

#### Union of Literals (Simplest)

```typescript
type Direction = 'up' | 'down' | 'left' | 'right';

const dir: Direction = 'up'; // OK
// const dir2: Direction = 'north'; // Error
```

#### Comparison Table

| Feature | Numeric Enum | String Enum | Const Enum | `as const` | Union |
|---------|--------------|-------------|------------|------------|-------|
| **Runtime code** | Yes (with reverse mapping) | Yes | No (inlined) | Yes (frozen object) | No |
| **Tree-shaking** | Poor | Moderate | Excellent | Excellent | Excellent |
| **`isolatedModules`** | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes | ✅ Yes |
| **Reverse mapping** | ✅ Yes | ❌ No | ✅ Yes (inlined) | ❌ No | ❌ No |
| **Iteration** | ✅ Yes | ✅ Yes | ✅ Yes (inlined) | ✅ Yes | ❌ No |
| **Readonly** | ⚠️ Mutable | ⚠️ Mutable | ⚠️ Mutable | ✅ Readonly | ✅ Readonly |
| **Recommendation** | ❌ Avoid | ⚠️ Use with care | ❌ Avoid | ✅ **Preferred** | ✅ **Preferred** |

#### Syntax Rules

- **Prefer `as const` objects** over enums for modern projects.
- **Prefer union types** for simple sets of literals.
- **Avoid numeric enums** — they have reverse mapping and are not tree-shakeable.
- **Avoid const enums** — they are incompatible with `isolatedModules`.
- **Use string enums** only if you need runtime iteration.
- **Use `as const` with `typeof`** to derive union types.
- **Use `satisfies`** to validate `as const` objects against a type.
- **Use `Object.values()`** to iterate over `as const` objects.

#### Constraints and Limitations

- **Numeric enums** produce reverse-mapped objects and extra code.
- **Const enums** are incompatible with `isolatedModules` and Babel.
- **Enums are not tree-shakeable** — the entire enum object is included.
- **Enums are not erased** — they exist at runtime.
- **`as const` objects are mutable at runtime** — TypeScript only enforces readonly at compile time.
- **`as const` is a compile-time assertion** — it does not prevent runtime mutation.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Enums vs. Const Assertions

```typescript
// enums-vs-const.ts
// ❌ Numeric enum — generates reverse mapping, not tree-shakeable
enum NumericStatus {
  Pending,
  Active,
  Inactive,
}
console.log(NumericStatus.Pending); // 0
console.log(NumericStatus[0]);      // 'Pending' — reverse mapping

// ⚠️ String enum — no reverse mapping, but not tree-shakeable
enum StringStatus {
  Pending = 'pending',
  Active = 'active',
  Inactive = 'inactive',
}
console.log(StringStatus.Pending);  // 'pending'
// console.log(StringStatus['pending']); // undefined — no reverse mapping

// ✅ Const assertion — zero-cost, tree-shakeable
const Status = {
  Pending: 'pending',
  Active: 'active',
  Inactive: 'inactive',
} as const;

type Status = typeof Status[keyof typeof Status];
// type Status = 'pending' | 'active' | 'inactive'

function handleStatus(status: Status): string {
  switch (status) {
    case Status.Pending: return 'Waiting...';
    case Status.Active: return 'Running...';
    case Status.Inactive: return 'Stopped.';
    default:
      const _exhaustive: never = status;
      throw new Error(`Unhandled status: ${_exhaustive}`);
  }
}

console.log(handleStatus(Status.Pending));  // 'Waiting...'
console.log(handleStatus(Status.Active));   // 'Running...'
console.log(handleStatus(Status.Inactive)); // 'Stopped.'

// ✅ Union of literals — simplest
type Direction = 'up' | 'down' | 'left' | 'right';

const move = (dir: Direction): string => `Moving ${dir}`;
console.log(move('up')); // 'Moving up'

// ✅ `satisfies` operator — validates const object against a type
const Routes = {
  home: '/',
  about: '/about',
  contact: '/contact',
} as const satisfies Record<string, `/${string}`>;

type Route = typeof Routes[keyof typeof Routes];
// type Route = '/' | '/about' | '/contact'
```

**Expected Output:**
```
0
Pending
pending
Waiting...
Running...
Stopped.
Moving up
```

**Why this works:** Numeric enums produce reverse mapping (0 ↔ 'Pending'). String enums produce a simple object. `as const` produces a readonly literal object with zero-cost types. The `satisfies` operator validates the object against a type without widening its type.

### Real-World Cases

- **API status codes:** `const StatusCode = { OK: 200, NotFound: 404 } as const`.
- **Configuration keys:** `const ConfigKeys = { DB_URL: 'DATABASE_URL' } as const`.
- **Feature flags:** `const Features = { NEW_UI: 'new_ui' } as const`.
- **Route names:** `const Routes = { HOME: '/', ABOUT: '/about' } as const`.
- **Event names:** `const Events = { USER_CREATED: 'user.created' } as const`.

---

## References

- TypeScript Documentation — Handbook — https://www.typescriptlang.org/docs/handbook/intro.html
- TypeScript Documentation — Everyday Types — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- TypeScript Documentation — Object Types — https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Documentation — Interfaces — https://www.typescriptlang.org/docs/handbook/2/objects.html#interfaces
- TypeScript Documentation — Type Aliases — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases
- TypeScript Documentation — Unions and Intersections — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types
- TypeScript Documentation — Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Documentation — Enums — https://www.typescriptlang.org/docs/handbook/enums.html
- TypeScript Documentation — `const` Assertions — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- TypeScript Documentation — `satisfies` Operator — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html
- TypeScript Documentation — tsconfig Reference — https://www.typescriptlang.org/tsconfig
- TypeScript Documentation — Strict Mode — https://www.typescriptlang.org/tsconfig#strict
- TypeScript Documentation — `strictNullChecks` — https://www.typescriptlang.org/tsconfig#strictNullChecks
- TypeScript Documentation — `noImplicitAny` — https://www.typescriptlang.org/tsconfig#noImplicitAny
- TypeScript Documentation — `noUncheckedIndexedAccess` — https://www.typescriptlang.org/tsconfig#noUncheckedIndexedAccess
- TypeScript Documentation — `exactOptionalPropertyTypes` — https://www.typescriptlang.org/tsconfig#exactOptionalPropertyTypes
- TypeScript Documentation — Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Documentation — Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Documentation — Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript Documentation — Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript GitHub — https://github.com/microsoft/TypeScript
- TypeScript Playground — https://www.typescriptlang.org/play
- Total TypeScript — https://www.totaltypescript.com/
- Matt Pocock — TypeScript Tips — https://www.totaltypescript.com/tips
- TypeScript Deep Dive — https://basarat.gitbook.io/typescript/
- DefinitelyTyped — https://github.com/DefinitelyTyped/DefinitelyTyped
- Type Challenges — https://github.com/type-challenges/type-challenges
- Zod — TypeScript-first Schema Validation — https://zod.dev/
- ts-reset — TypeScript Reset — https://github.com/total-typescript/ts-reset