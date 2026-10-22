# Union and Intersection Patterns in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Union and intersection types** are the two fundamental **type composition operators** in TypeScript’s type algebra. Union types (`A | B`) represent values that belong to **at least one** of several constituent types (OR semantics). Intersection types (`A & B`) represent values that belong to **all** constituent types simultaneously (AND semantics). **Union and intersection patterns** are the recurring idioms—discriminated unions, exhaustive switches, mixin compositions, branded types, and state machines—that apply these operators to model complex, type-safe data structures.

**Technical Definition.** Union types are the **set-theoretic union** of their constituents: `A | B` accepts any value assignable to `A` or `B`. Intersection types are the **set-theoretic intersection**: `A & B` accepts only values assignable to both `A` and `B`, with object properties recursively merged. Union patterns leverage **control-flow narrowing** (via `typeof`, `instanceof`, `in`, equality, and discriminants) to refine unions in branches. Intersection patterns leverage **composition** to combine independent type pieces. Combined patterns interleave both operators, e.g., `Base & (A | B)`, to express “this AND (that OR the other).” All union and intersection types are **erased** during compilation.

**Beginner-Friendly Explanation.** Union and intersection types are **the two ways to combine types in TypeScript**. A union (`|`) is an **OR**: “this value is a string OR a number.” An intersection (`&`) is an **AND**: “this value must have everything from type A AND everything from type B.” Union patterns are the **techniques** for working with OR-types safely—like checking which variant you have before using it. Intersection patterns are the **techniques** for composing AND-types—like gluing small type pieces together to make a richer type. Combined patterns use both together when a value must be one thing **and** one of several other things.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **OR vs. AND** | `\|` means “one of these types”; `&` means “all of these types.” |
| **Narrowing required for unions** | Only common members are accessible on a union until narrowed. |
| **Recursive merging for intersections** | Intersections recursively merge object properties on every use. |
| **Silent `never` on intersection conflicts** | Incompatible properties in an intersection produce `never` without a declaration-time error. |
| **Discriminated unions** | A literal-typed discriminant enables automatic narrowing and exhaustiveness checking. |
| **Performance asymmetry** | Intersections are recomputed on every use; interfaces are cached. Large unions (>12–50 members) cause quadratic checking. |
| **Erasure** | Both operators exist only at compile time and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type aliases, interfaces, and object types.
- **Literal types**: `"idle"`, `42`, `true`.
- **Control flow**: `switch`, `if`/`else`, and early returns.
- **Type narrowing and type guards**.
- **Generics fundamentals**: type parameters and constraints.

---

### Related Programming Areas

- **Discriminated unions** and **exhaustiveness checking**.
- **Mixin patterns** (intersection of constructor types).
- **Branded/nominal types** (intersection with phantom markers).
- **State machine modeling** (union of state variants).
- **Compiler performance optimization** (interfaces over intersections, base types over large unions).
- **Algebraic data types** (sum types = unions; product types = intersections).

---

## Core Concepts / Features

---

## 1. Union Patterns

### Definitions

**Core Definition.** **Union patterns** are the recurring idioms for declaring, narrowing, and exhaustively handling union types in TypeScript, including discriminated unions, exhaustive `switch` statements, early-return narrowing, and state-machine modeling.

**Technical Definition.** Union patterns leverage **control-flow analysis** to refine a union to a specific member within a guarded branch. The primary patterns are: **discriminated unions** (common literal-typed discriminant + `switch`); **exhaustiveness checking** (`never`-typed `default` branch); **early-return narrowing** (`in` checks + `return`); and **literal unions for closed sets** (preferred over `enum`). Narrowing propagates through `if`, `switch`, ternaries, logical operators, and early returns.

**Beginner-Friendly Explanation.** Union patterns are **the standard ways people work with OR-types safely**. The most important is the **discriminated union**: give each variant a label (the discriminant), then `switch` on that label so TypeScript knows which variant you have. Always handle every case and put a `never` check in the `default` branch so you get a compile-time error if you forget one.

### Purposes

- **To model mutually exclusive variants** with precise, type-safe shapes.
- **To enable automatic narrowing** without unsafe type assertions.
- **To enforce exhaustiveness** so no variant is forgotten when new ones are added.
- **To make invalid states unrepresentable** by giving each variant only its relevant fields.
- **To replace optional-field “god objects”** that allow illegal combinations.

### Syntax Rules and Structure

**Discriminated union syntax:**

```
type Union =
  | { discriminant: "value1"; /* fields for variant 1 */ }
  | { discriminant: "value2"; /* fields for variant 2 */ };
```

**Exhaustive `switch` with `never`:**

```
function handle(value: Union): ReturnType {
  switch (value.discriminant) {
    case "value1": /* narrowed to variant 1 */ break;
    case "value2": /* narrowed to variant 2 */ break;
    default: {
      const unreachable: never = value;
      return unreachable;
    }
  }
}
```

**Component breakdown:**

- `discriminant` — the shared literal-typed property.
- `"value1"`, `"value2"` — unique literal values per variant.
- `default` branch — receives `never` when all variants are handled.
- `unreachable: never` — compile-time exhaustiveness check.

**Syntax rules.**

- The discriminant must be a **literal type** (string, number, `null`, or `undefined`).
- Every member must declare the discriminant with a **unique value**.
- `switch` on the discriminant is the idiomatic narrowing pattern.
- The `default` branch should assign the value to `never` or call an `assertNever` helper.
- Early returns can narrow the remaining type without a `switch`.

**Constraints and limitations.**

- **Methods cannot be discriminants**; only literal-typed properties work.
- **Nested discriminants do not narrow automatically**; the discriminant must be at the same level.
- **Large unions** (50+ members) cause quadratic checking; group them into categories.
- The `never` check must be in a `default` branch (or a final `else`).

### Annotated Complete Step by Step Code Examples

#### Example 1: Discriminated union with exhaustive `switch`

```typescript
// example_union_pattern.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "triangle"; base: number; height: number };

function assertNever(value: never): never {
  throw new Error(`Unhandled: ${JSON.stringify(value)}`);
}

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2; // narrowed to Circle
    case "square":
      return shape.side ** 2; // narrowed to Square
    case "triangle":
      return (shape.base * shape.height) / 2; // narrowed to Triangle
    default:
      return assertNever(shape); // ✅ compiles: all variants handled
  }
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2)); // Expected: 3.14
console.log(area({ kind: "square", side: 2 })); // Expected: 4
```

**Expected Output:**
```
3.14
4
```

**Why this output occurs.** The `kind` discriminant is a literal type per variant. The `switch` narrows `shape` to the matching member, enabling access to `radius`, `side`, or `base`/`height`. After all cases, `shape` in `default` has type `never`, so `assertNever` accepts it. Adding a new variant without handling it causes a compile-time error.

---

#### Example 2: Early-return narrowing with `in`

```typescript
// example_union_early_return.ts
type APIResponse =
  | { version: 0; msg: string }
  | { version: 1; message: string; status: number }
  | { error: string };

function handle(response: APIResponse): void {
  if ("error" in response) {
    console.error(response.error);
    return; // Excludes the error member.
  }

  if (response.version === 0) {
    console.log(response.msg); // narrowed to version 0
  } else {
    console.log(response.status, response.message); // narrowed to version 1
  }
}

handle({ error: "Not found" }); // Expected: Not found
handle({ version: 0, msg: "OK" }); // Expected: OK
```

**Expected Output:**
```
Not found
OK
```

**Why this output occurs.** The `in` check narrows the union, and the early `return` excludes the `error` member from the rest of the function. The remaining code only deals with the versioned members, which are then narrowed by `version` checks.

---

#### Example 3: State machine with discriminated union

```typescript
// example_union_state_machine.ts
type Job =
  | { kind: "pending" }
  | { kind: "running"; startedAt: Date }
  | { kind: "done"; startedAt: Date; result: Uint8Array }
  | { kind: "failed"; startedAt: Date; error: string };

function describe(job: Job): string {
  switch (job.kind) {
    case "pending": return "queued";
    case "running": return `running since ${job.startedAt.toISOString()}`;
    case "done": return "finished";
    case "failed": return job.error;
    default: {
      const unreachable: never = job;
      return unreachable;
    }
  }
}

console.log(describe({ kind: "pending" })); // Expected: queued
```

**Expected Output:**
```
queued
```

**Why this output occurs.** Each state carries exactly the fields relevant to it—`startedAt` only appears in non-pending states, `result` only in `done`, and `error` only in `failed`. Invalid states (e.g., `pending` with a `result`) are unrepresentable. The `never` check ensures exhaustiveness.

---

### Real-World Cases

- **React data fetching**: `FetchState<T> = { status: "idle" } | { status: "loading" } | { status: "success"; data: T } | { status: "error"; error: Error }`.
- **Redux reducers**: `switch (action.type)` with `assertNever` in the default case.
- **Job queues**: `pending | running | done | failed` with state-specific fields.
- **Form validation**: `{ status: "valid"; value: string } | { status: "invalid"; errors: string[] }`.

---

## 2. Intersection Patterns

### Definitions

**Core Definition.** **Intersection patterns** are the recurring idioms for composing types with the `&` operator, including interface composition, mixin patterns (intersecting constructor types), branded types (intersecting a base type with a phantom marker), and one-off utility combinations.

**Technical Definition.** Intersection patterns leverage **recursive property merging** to combine independent type pieces into a richer type. The primary patterns are: **interface composition** (`type Response = Data & ErrorHandling`); **mixin patterns** (intersecting a mixin constructor type with a base class constructor type); **branded types** (`string & { __brand: "UserId" }`); and **utility type composition** (`Omit<T, K> & { extra: string }`). Intersections do **not** detect property conflicts at declaration; incompatible properties reduce to `never`.

**Beginner-Friendly Explanation.** Intersection patterns are **the standard ways people glue types together with AND**. The most important are: combining small interfaces into richer ones (composition), creating mixins that add behavior to classes, and branding primitives so a `UserId` is not interchangeable with a plain `string`.

### Purposes

- **To compose multiple types** into one that satisfies all of them.
- **To extend type aliases** that cannot use `extends` (unions, primitives, functions).
- **To implement mixins** by intersecting constructor types.
- **To create branded/nominal types** for domain safety.
- **To combine utility types** for precise one-off types.

### Syntax Rules and Structure

**Interface composition:**

```
type Response = Data & ErrorHandling;
```

**Mixin pattern:**

```
type Constructor = new (...args: any[]) => {};
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base { timestamp = new Date(); };
}
```

**Branded type:**

```
type UserId = string & { __brand: "UserId" };
```

**Component breakdown:**

- `&` — intersection operator.
- Constituents — interfaces, type aliases, object literals, primitives, function types.

**Syntax rules.**

- `&` has **higher precedence** than `|`: `A & B | C` is `(A & B) | C`.
- Intersections may combine any types: objects, primitives, functions, tuples.
- Duplicate members are merged; **conflicting members become `never`** without a declaration-time error.
- Mixin patterns require a constructor type constraint (`new (...args: any[]) => {}`).

**Constraints and limitations.**

- **Silent `never`**: incompatible properties reduce to `never` without a declaration-time error, unlike interface `extends`.
- **Performance cost**: intersections are **recomputed on every use**, unlike cached interfaces.
- Intersections cannot express unions or conditional types directly.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface composition with intersection

```typescript
// example_intersection_composition.ts
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
}

interface ArtworksData {
  artworks: { title: string }[];
}

type ArtworksResponse = ArtworksData & ErrorHandling;

const response: ArtworksResponse = {
  success: true,
  artworks: [{ title: "Mona Lisa" }],
};

console.log(response.artworks[0].title); // Expected: Mona Lisa
console.log(response.success); // Expected: true
```

**Expected Output:**
```
Mona Lisa
true
```

**Why this output occurs.** `ArtworksResponse` requires all members of both `ArtworksData` and `ErrorHandling`. The object literal satisfies both contracts. This is the canonical composition pattern from the TypeScript Playground.

---

#### Example 2: Mixin pattern with constructor intersection

```typescript
// example_intersection_mixin.ts
type Constructor = new (...args: any[]) => {};

function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActive = false;
    activate() { this.isActive = true; }
    deactivate() { this.isActive = false; }
  };
}

class User {
  constructor(public name: string) {}
}

const ActiveUser = Activatable(User);
const user = new ActiveUser("Ada");

user.activate();
console.log(user.isActive); // Expected: true
console.log(user.name); // Expected: Ada
```

**Expected Output:**
```
true
Ada
```

**Why this output occurs.** The mixin pattern in TypeScript 2.2+ represents a mixin class as an **intersection** between the mixin constructor type and the base class constructor type. The resulting `ActiveUser` type combines `User`’s members with the mixin’s members.

---

#### Example 3: Branded type with primitive intersection

```typescript
// example_intersection_branded.ts
type UserId = string & { __brand: "UserId" };

function createUserId(id: string): UserId {
  return id as UserId;
}

const userId = createUserId("u1");
console.log(typeof userId); // Expected: string

// const plain: string = userId; // ✅ assignable
// const wrong: UserId = "plain"; // ❌ Error: string not assignable to UserId
```

**Expected Output:**
```
string
```

**Why this output occurs.** The intersection combines `string` with a phantom brand property. At runtime, the value is a plain string; the brand exists only in the type system. This prevents accidentally passing a plain string where a `UserId` is expected. Interfaces cannot extend primitives, so intersections are mandatory here.

---

### Real-World Cases

- **API response composition**: `type ArtworksResponse = ArtworksData & ErrorHandling`.
- **React prop composition**: `type ButtonProps = BaseProps & StyledProps & A11yProps`.
- **Branded domain IDs**: `type UserId = string & { __brand: "UserId" }`.
- **Mixin libraries**: `Activatable(User)` returning a class with combined behavior.

---

## 3. Combined Patterns

### Definitions

**Core Definition.** **Combined patterns** interleave union and intersection operators—such as `Base & (A | B)` or `(A & B) | C`—to express constraints where a value must satisfy a **base contract AND one of several variants**, or must match **one of several composed shapes**.

**Technical Definition.** Combined patterns leverage the **precedence** of `&` over `|` to build precise types. The canonical pattern is `Base & (VariantA | VariantB)`: the value must have all of `Base`’s members **and** exactly one of the variant shapes. Another pattern is `(A & B) | (C & D)`: the value is either the `A & B` composition or the `C & D` composition. Combined patterns are essential for modeling **requests with a common envelope and variable payloads**, **form fields with base metadata and one-of value types**, and **discriminated unions whose members are themselves intersections**.

**Beginner-Friendly Explanation.** Combined patterns are **using OR and AND together**. For example, “this object must have a `userId` AND it must be either an `{ html: string }` OR a `{ markdown: string }`.” It lets you say “this AND (that OR the other)” in a single type.

### Purposes

- **To express “base contract + one-of variants”** in a single type.
- **To model requests with a common envelope and variable payloads**.
- **To compose discriminated union members** from intersections.
- **To create precise utility types** that combine `Omit<T, K>` with one-of unions.

### Syntax Rules and Structure

**Base & (A | B) pattern:**

```
type Request = Base & ({ html: string } | { markdown: string });
```

**Precedence note:**

```
type Combined = A & B | C; // Parsed as (A & B) | C
```

**Component breakdown:**

- `Base` — the required common fields.
- `(A | B)` — the one-of variants; parentheses are required to override precedence.
- `&` binds tighter than `|`; use parentheses for clarity.

**Syntax rules.**

- Parentheses are **required** to group unions under intersections: `Base & (A | B)`.
- Without parentheses, `Base & A | B` parses as `(Base & A) | B`.
- Combined patterns may be nested: `Base & (A | (B & C))`.
- Combined patterns work with discriminated unions when each intersection member has a discriminant.

**Constraints and limitations.**

- Combined patterns increase type complexity and may slow compilation.
- Debugging combined types can be harder because error messages expand the full expression.
- Intersection conflicts inside a union variant reduce to `never` silently.

### Annotated Complete Step by Step Code Examples

#### Example 1: Base & (A | B) pattern

```typescript
// example_combined_base_or.ts
interface CreateArtistBioBase {
  artistID: string;
  thirdParty?: boolean;
}

type CreateArtistBioRequest =
  CreateArtistBioBase & ({ html: string } | { markdown: string });

const workingRequest: CreateArtistBioRequest = {
  artistID: "banksy",
  markdown: "Banksy is an anonymous graffiti artist...",
};

console.log(workingRequest.artistID); // Expected: banksy

// const badRequest: CreateArtistBioRequest = {
//   artistID: "banksy",
//   // ❌ Error: missing html or markdown
// };
```

**Expected Output:**
```
banksy
```

**Why this output occurs.** The intersection requires the base fields **and** exactly one of `html` or `markdown`. The object provides `artistID` and `markdown`, satisfying both the intersection and the union inside it. This is the canonical combined pattern from the TypeScript Playground.

---

#### Example 2: Discriminated union with intersection members

```typescript
// example_combined_discriminated.ts
interface BaseEvent {
  timestamp: Date;
  userId: string;
}

type UserEvent = BaseEvent & { kind: "login"; method: string };
type PageEvent = BaseEvent & { kind: "view"; url: string };

type AppEvent = UserEvent | PageEvent;

function handle(event: AppEvent): void {
  switch (event.kind) {
    case "login":
      console.log(`Login via ${event.method}`); // narrowed to UserEvent
      break;
    case "view":
      console.log(`View ${event.url}`); // narrowed to PageEvent
      break;
  }
}

handle({
  kind: "login",
  method: "password",
  timestamp: new Date(),
  userId: "u1",
}); // Expected: Login via password
```

**Expected Output:**
```
Login via password
```

**Why this output occurs.** Each union member is an **intersection** of `BaseEvent` and a specific variant. The `kind` discriminant narrows the union to the matching intersection, enabling access to `method` or `url`. The base fields (`timestamp`, `userId`) are available on both.

---

### Real-World Cases

- **API request payloads**: `BaseRequest & ({ body: string } | { json: object })`.
- **Form field states**: `BaseField & ({ status: "valid"; value: string } | { status: "invalid"; errors: string[] })`.
- **Discriminated unions with shared metadata**: `BaseEvent & { kind: "login" }` and `BaseEvent & { kind: "view" }`.

---

## 4. Pattern Best Practices

### Definitions

**Core Definition.** **Union and intersection pattern best practices** are guidelines for choosing between unions and intersections, structuring them for **type safety, compiler performance, and maintainability**, and avoiding the pitfalls of silent `never` conflicts, quadratic union checking, and non-exhaustive handling.

**Technical Definition.** Best practices center on **modeling invalid states as impossible** (discriminated unions over optional fields), **always using exhaustive `switch` with `never`**, **preferring interface `extends` over intersections for object hierarchies** (caching, conflict detection, 2–3× faster type-checking), **avoiding large flat unions** (>12–50 members cause quadratic checking), **using literal unions over `enum`**, **keeping discriminants at the top level**, and **using intersections only for primitives, functions, mixins, and one-off combinations**.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep union and intersection patterns safe and fast. The big rules are: use discriminated unions for variants; always handle every case with a `never` check; use `interface extends` instead of `&` for object hierarchies; and do not create enormous flat unions—group them into categories.

### Purposes

- **To make invalid states unrepresentable**, eliminating a class of runtime bugs.
- **To catch missing cases at compile time** when new variants are added.
- **To maximize compiler performance** by preferring cached interfaces over recomputed intersections and avoiding quadratic unions.
- **To choose the right tool** between union, intersection, and interface `extends`.
- **To keep type definitions readable and maintainable**.

### Syntax Rules and Structure

Best practices are conventions that interact with union and intersection syntax. The core rules are:

1. **Model one variant per legal state.** Use discriminated unions instead of optional-field objects that allow illegal combinations.
2. **Always use exhaustive `switch` with `never` in the default branch.** This ensures compile-time errors when new variants are added but not handled.
3. **Prefer interface `extends` over intersections for object hierarchies.** Interfaces create a single flat type, detect property conflicts at declaration, and are **cached by the compiler**—resulting in **2–3× faster type-checking**.
4. **Avoid large flat unions.** Unions with **>12 elements** cause quadratic checking; **>50 members** cause measurable compilation slowdowns. Group into discriminated categories or use base types.
5. **Use literal unions, not `enum`, for closed sets.** String literal unions have zero runtime cost, serialize as themselves, and avoid the pitfalls of numeric enums.
6. **Keep the discriminant at the top level.** Nested discriminants do not narrow automatically.
7. **Use intersections for primitives, functions, mixins, and one-off combinations.** These are the cases where interface `extends` cannot work.

**Component breakdown (pattern decision tree):**

```
Do you have mutually exclusive variants?
  ├── Yes → Use a discriminated union + exhaustive switch.
  └── No → Are you composing object types used repeatedly?
           ├── Yes → Use interface extends (cached, conflict-detecting).
           └── No → Are you combining primitives, functions, or one-off types?
                    ├── Yes → Use intersection (&).
                    └── No → Re-evaluate; interface extends is likely better.
```

**Constraints and limitations.**

- Discriminated unions require literal-typed discriminants; methods and computed properties do not work.
- Intersections silently produce `never` on conflicts; interface `extends` errors at declaration.
- Large unions may be unavoidable for generated types; mitigate with grouping or branding.

### Annotated Complete Step by Step Code Examples

#### Example 1: Discriminated union over optional fields

```typescript
// example_best_discriminated_over_optional.ts
// ❌ Optional fields allow impossible combinations.
// interface Job {
//   status: "pending" | "running" | "done" | "failed";
//   startedAt?: Date;
//   result?: Uint8Array;
//   error?: string;
// }

// ✅ Discriminated union: one variant per legal state.
type Job =
  | { kind: "pending" }
  | { kind: "running"; startedAt: Date }
  | { kind: "done"; startedAt: Date; result: Uint8Array }
  | { kind: "failed"; startedAt: Date; error: string };

function assertNever(value: never): never {
  throw new Error(`Unhandled: ${JSON.stringify(value)}`);
}

function describe(job: Job): string {
  switch (job.kind) {
    case "pending": return "queued";
    case "running": return `running since ${job.startedAt.toISOString()}`;
    case "done": return "finished";
    case "failed": return job.error;
    default: return assertNever(job);
  }
}

console.log(describe({ kind: "pending" })); // Expected: queued
```

**Expected Output:**
```
queued
```

**Why this output occurs.** Each state carries only its relevant fields. The invalid state `{ kind: "pending", result: ... }` is rejected by the compiler. The `never` check ensures exhaustiveness.

---

#### Example 2: Interface `extends` over intersection for object hierarchies

```typescript
// example_best_extends_over_intersection.ts
// ❌ Intersection: recomputed on every use, no conflict detection.
// type UserWithPermissions = User & Permissions & AuditInfo;

// ✅ Interface extends: cached, conflict-detecting, 2–3× faster.
interface User { id: string; name: string; }
interface Permissions { canRead: boolean; canWrite: boolean; }
interface AuditInfo { createdAt: Date; }

interface UserWithPermissions extends User, Permissions, AuditInfo {}

const admin: UserWithPermissions = {
  id: "u1",
  name: "Ada",
  canRead: true,
  canWrite: true,
  createdAt: new Date(),
};

console.log(admin.canWrite); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The interface hierarchy uses `extends`, which creates a single flat type, detects conflicts at declaration, and is cached by the compiler. The intersection version would recursively merge on every use, causing slower type-checking.

---

#### Example 3: Grouping large unions into discriminated categories

```typescript
// example_best_grouping.ts
// ❌ 200+ flat union members cause quadratic checking.
// ✅ Group into discriminated categories.
type UserEvent = { category: "user"; action: "login" | "logout" | "signup" };
type PageEvent = { category: "page"; action: "view" | "scroll" | "leave" };
type FormEvent = { category: "form"; action: "submit" | "validate" | "reset" };

type AppEvent = UserEvent | PageEvent | FormEvent;

function handleEvent(event: AppEvent): void {
  switch (event.category) {
    case "user":
      console.log(`User action: ${event.action}`); // narrowed to UserEvent
      break;
    case "page":
      console.log(`Page action: ${event.action}`); // narrowed to PageEvent
      break;
    case "form":
      console.log(`Form action: ${event.action}`); // narrowed to FormEvent
      break;
  }
}

handleEvent({ category: "user", action: "login" }); // Expected: User action: login
```

**Expected Output:**
```
User action: login
```

**Why this output occurs.** The outer union has only **three members**, avoiding quadratic checking. Each category narrows to its specific `action` union. This pattern keeps compiler performance reasonable while preserving type safety.

---

### Real-World Cases

- **React state management**: `FetchState<T>` with exhaustive rendering.
- **Enterprise codebases**: interface `extends` for object hierarchies to benefit from caching and conflict detection.
- **Analytics events**: grouping large event unions into discriminated categories.
- **Domain modeling**: literal unions for closed sets (`type Status = "idle" | "loading" | "success"`).

---

## References

- TypeScript Playground — Union and Intersection Types - https://www.typescriptlang.org/play/typescript/primitives/union-and-intersection-types.ts.html
- TypeScript Playground — Discriminate Types - https://www.typescriptlang.org/play/typescript/meta-types/discriminate-types.ts.html
- TypeScript Handbook — Narrowing: Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook — Unions and Intersection Types (Deprecated) - https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html
- TypeScript Performance Wiki — Preferring Interfaces Over Intersections - https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections
- TypeScript Performance Wiki — Preferring Base Types Over Unions - https://github.com/microsoft/TypeScript/wiki/Performance#preferring-base-types-over-unions
- TypeScript 2.2 Release Notes — Mixin Class Pattern (Intersection Types) - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-2.html
- Steve Kinney — Unions, Intersections, and Type Guards - https://stevekinney.com/courses/react-typescript/typescript-unions-intersections-guards
- Steve Kinney — Scaling TypeScript (Performance) - https://stevekinney.com/courses/enterprise-ui/scaling-typescript
- OneUptime Blog — How to Handle Union and Intersection Types - https://github.com/oneuptime/blog/tree/master/posts/2026-01-24-typescript-union-intersection-types
- TypeScript ESLint — `switch-exhaustiveness-check` Rule - https://typescript-eslint.io/rules/switch-exhaustiveness-check/