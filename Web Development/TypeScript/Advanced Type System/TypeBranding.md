# Type Branding in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Type branding** is a user-land TypeScript pattern that simulates **nominal typing** within TypeScript's structurally typed system by **intersecting a base type with a phantom, type-only marker**. The resulting **branded type**—also called an **opaque type** or **nominal type**—is structurally distinct from other types sharing the same runtime representation, preventing accidental interchange of semantically different values without any runtime overhead.

**Technical Definition.** Type branding uses a **generic utility type** `Brand<T, B>` defined as `T & { readonly [brand]: B }`, where `brand` is a `unique symbol` and `B` is a string literal tag. The brand property **exists only at the type level** and is never assigned at runtime; it is a **phantom property**. Because TypeScript's assignability rules treat the unique symbol–keyed property as a required member, two branded types with different tags are **mutually incompatible** even when their base types are identical. Branding is **compile-time only** and is **erased** during compilation. Values are converted from their base type to the branded type through **gatekeeper functions** that validate and cast once, centralizing the only `as` assertion in the codebase.

**Beginner-Friendly Explanation.** TypeScript has **structural typing**: two types are compatible if they have the same shape. That means `type UserId = string` and `type OrderId = string` are **the same type** to the compiler—you can accidentally pass an order ID where a user ID is expected, and TypeScript will not complain. **Type branding** fixes this by attaching an **invisible label** to the type. The label does not exist at runtime—it is purely a compile-time marker—but it makes `UserId` and `OrderId` **incompatible** so the compiler catches the swap. Think of it as giving each type a **unique passport** that only the type system can see.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Nominal emulation** | Creates nominally distinct types within a structural system. |
| **Phantom property** | The brand marker exists only in the type system; it has **no runtime representation**. |
| **Zero runtime overhead** | Brands are erased during compilation and add no runtime cost. |
| **Unique symbol keys** | The recommended brand key is a `unique symbol`, preventing accidental collision and replication. |
| **Gatekeeper construction** | Values are converted from base types through validated factory functions that centralize the only cast. |
| **Compile-time only** | Brands cannot be inspected at runtime; they are purely a type-level pattern. |
| **Forgeable with `as`** | Brands are a user-land pattern and can be bypassed with type assertions; discipline is required. |
| **Performance-sensitive** | Complex branded types can trigger heavy type inference; keep brands simple. |

---

### Prerequisites

- **TypeScript basics**: type aliases, interfaces, intersection types (`&`), and generics.
- **Structural vs. nominal typing**: understanding why TypeScript allows structurally identical types to be interchanged.
- **`unique symbol`**: how to declare and use unique symbols as property keys.
- **Type assertions (`as`)**: their purpose and risks.
- **Runtime validation**: functions that check values before casting.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Nominal typing** — the type-system discipline that branding emulates.
- **Opaque types** — the general term for types whose underlying representation is hidden.
- **Phantom types** — types with type-level-only parameters or properties.
- **Domain-Driven Design (DDD)** — branded types enforce domain integrity for identifiers and value objects.
- **Runtime validation** — libraries like Zod, `io-ts`, and `arktype` integrate with branded types.
- **API design** — branded IDs prevent argument-swap bugs in service boundaries.

---

## Core Concepts / Features

---

## 1. Type Branding

### Definitions

**Core Definition.** **Type branding** is the technique of intersecting a base type with a **phantom marker property** to create a **nominally distinct type** that is incompatible with other types sharing the same base representation.

**Technical Definition.** Branding exploits TypeScript's **intersection types** and **`unique symbol`** keys to add a required member that exists only in the type system. The branded type `T & { readonly [brand]: B }` is a subtype of `T`—assignable **to** `T`—but `T` is **not assignable to** the branded type because it lacks the phantom property. This creates **one-way nominality**: branded values can flow into base-type positions, but raw base values cannot flow into branded positions without explicit construction. The brand is **erased** and has no runtime footprint.

**Beginner-Friendly Explanation.** Type branding is **sticking an invisible sticker on a type** so the compiler can tell it apart from look-alikes. The sticker never appears at runtime, but TypeScript treats it as real. You can use a branded value anywhere the base type is expected, but you cannot use a raw value where the branded type is required.

### Purposes

- **To prevent accidental interchange of structurally identical values** such as `UserId`, `OrderId`, and `ProductId`, all of which are strings at runtime.
- **To model validated data**—emails, URLs, UUIDs, positive integers—so that only validated values can reach downstream code.
- **To enforce domain invariants at compile time** without runtime wrappers or class instances.
- **To distinguish units of measure** (`Meters` vs. `Seconds`, `USD` vs. `EUR`) that share the same numeric representation.
- **To centralize validation at boundaries** through gatekeeper functions, following the “parse, don’t validate” principle.
- **To enable safe refactoring** by turning naming conventions into compiler-enforced contracts.

### Syntax Rules and Structure

**Core brand utility:**

```typescript
declare const __brand: unique symbol;

type Brand<T, B extends string> = T & { readonly [__brand]: B };
```

**Branded type declarations:**

```typescript
type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;
type Cents = Brand<number, "Cents">;
```

**Gatekeeper constructor:**

```typescript
function createUserId(id: string): UserId {
  if (!/^user_/.test(id)) throw new Error("Invalid UserId");
  return id as UserId; // The only permitted cast.
}
```

**Component breakdown:**

- `declare const __brand: unique symbol` — a type-only unique symbol used as the phantom key. It is never assigned a runtime value.
- `Brand<T, B extends string>` — the generic brand utility: intersects the base type `T` with a phantom property keyed by the unique symbol and tagged with `B`.
- `UserId = Brand<string, "UserId">` — a branded type with base `string` and tag `"UserId"`.
- `createUserId` — a gatekeeper function that validates and casts once.

**Syntax rules.**

- The brand key must be a **`unique symbol`** for strong nominality; string keys (`__brand`) are forgeable and can collide.
- The brand tag `B` should be a **string literal** unique to the type.
- The phantom property must be **`readonly`** to prevent mutation in type-level operations.
- Gatekeeper functions should be the **only place** where `as` casts occur.
- Branded types are **assignable to their base type**, but not vice versa.

**Constraints and limitations.**

- Brands are **erased at runtime**; they provide no runtime validation by themselves.
- Brands are **forgeable** with `as`; code review discipline and lint rules are required.
- Complex brands can cause **type inference performance issues**, especially with primitives in generic utilities.
- Brands cannot be used with `instanceof` or runtime reflection.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic branding with unique symbol

```typescript
// example_brand_basic.ts
// Step 1: Declare the brand key and utility.
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

// Step 2: Define branded types.
type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;

// Step 3: Create a gatekeeper function.
function createUserId(id: string): UserId {
  if (!id.startsWith("user_")) {
    throw new Error(`Invalid UserId: ${id}`);
  }
  return id as UserId;
}

// Step 4: Use the branded type.
function getOrder(userId: UserId, orderId: OrderId) {
  console.log(`Fetching order ${orderId} for user ${userId}`);
}

const uid = createUserId("user_123");
// getOrder(uid, "order_456"); // ❌ Error: string not assignable to OrderId.
console.log(uid); // Expected: user_123
```

**Setup.** Compile with `npx tsc example_brand_basic.ts --strict`.

**Expected Output:**
```
user_123
```

**Why this output occurs.** `UserId` is `string & { readonly [__brand]: "UserId" }`, and `OrderId` has the `"OrderId"` tag. The compiler treats them as incompatible despite both being strings at runtime. `createUserId` is the only place where the cast from `string` to `UserId` occurs.

---

#### Example 2: Branded numeric units

```typescript
// example_brand_units.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

type Meters = Brand<number, "Meters">;
type Seconds = Brand<number, "Seconds">;

function meters(n: number): Meters {
  return n as Meters;
}

function seconds(n: number): Seconds {
  return n as Seconds;
}

function speed(distance: Meters, time: Seconds): number {
  return distance / time;
}

console.log(speed(meters(100), seconds(10))); // Expected: 10

// speed(seconds(10), meters(100)); // ❌ Error: arguments swapped.
```

**Expected Output:**
```
10
```

**Why this output occurs.** `Meters` and `Seconds` are both numbers at runtime but nominally distinct at compile time. Swapping the arguments is a type error, preventing a unit-of-measure bug.

---

#### Example 3: Validated email with a type predicate

```typescript
// example_brand_predicate.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

type ValidatedEmail = Brand<string, "ValidatedEmail">;

function isValidEmail(value: string): value is ValidatedEmail {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
}

function sendEmail(email: ValidatedEmail) {
  console.log(`Sending to ${email}`);
}

const input: string = "ada@example.com";
if (isValidEmail(input)) {
  sendEmail(input); // ✅ narrowed to ValidatedEmail.
}
// sendEmail(input); // ❌ Error: string not assignable to ValidatedEmail.
```

**Expected Output:**
```
Sending to ada@example.com
```

**Why this output occurs.** The type predicate `value is ValidatedEmail` narrows `input` after the runtime check. The branded type ensures that only validated strings reach `sendEmail`.

---

### Real-World Cases

- **Domain identifiers**: `UserId`, `OrderId`, `ProductId`, `SessionId`, `TransactionId`.
- **Validated strings**: `EmailAddress`, `UrlString`, `HexColor`, `ISODateString`, `SanitizedHTML`.
- **Units of measure**: `Meters`, `Seconds`, `Cents`, `Percentage`, `USD`, `EUR`.
- **Security-sensitive values**: `SanitizedHTML`, `HashedPassword`, `AccessToken`.
- **API boundaries**: branded IDs prevent argument swaps in service calls and database queries.

---

## 2. Branded Types

### Definitions

**Core Definition.** A **branded type** is a type produced by intersecting a base type with a **phantom brand marker**, making it **nominally distinct** from other types with the same underlying representation.

**Technical Definition.** A branded type `Brand<T, B>` is technically `T & { readonly [__brand]: B }`, where `__brand` is a `unique symbol` and `B` is a string literal tag. It is a **subtype of `T`**: branded values are assignable to `T`, but `T` is not assignable to the branded type. Branded types may be created with **string literal keys** (`__brand`) or **unique symbol keys**; unique symbols are the recommended pattern because they cannot be accidentally replicated or collided across modules. The **tag** `B` provides the nominal identity: `Brand<string, "UserId">` and `Brand<string, "OrderId">` are mutually incompatible.

**Beginner-Friendly Explanation.** A branded type is **a type with a secret handshake**. It looks and behaves like a string or number at runtime, but the compiler knows it is special and will not let you mix it up with other look-alikes.

### Purposes

- **To create nominally distinct types from primitives** without runtime wrappers.
- **To carry proof of validation**—a `ValidatedEmail` proves that the value passed an email check.
- **To prevent argument-swap bugs** in functions with multiple same-typed parameters.
- **To model domain concepts** as first-class types rather than naming conventions.
- **To integrate with runtime validators** like Zod, which can infer branded types.
- **To enable safe parsing** at system boundaries, following “parse, don’t validate.”

### Syntax Rules and Structure

**Generic brand utility (recommended):**

```typescript
declare const __brand: unique symbol;

type Brand<T, B extends string> = T & { readonly [__brand]: B };
```

**Per-type unique symbol (strongest):**

```typescript
declare const UserIdBrand: unique symbol;
type UserId = string & { readonly [UserIdBrand]: "UserId" };
```

**String-key brand (simplest, weakest):**

```typescript
type UserId = string & { readonly __brand: "UserId" };
```

**Gatekeeper constructor pattern:**

```typescript
function createUserId(id: string): UserId {
  if (!id.startsWith("user_")) throw new Error("Invalid UserId");
  return id as UserId;
}
```

**Syntax rules.**

- Prefer a **single `unique symbol`** key shared across all brands via a generic utility.
- The tag `B` should be a **unique string literal** per branded type.
- The phantom property must be **`readonly`**.
- Construction should go through **gatekeeper functions** that validate and cast once.
- Branded types are **assignable to their base type**; the reverse requires construction.

**Constraints and limitations.**

- **String-key brands are forgeable** and can collide; unique symbols are preferred.
- Brands are **erased**; they provide no runtime validation.
- Brands can **worsen type inference performance** when used with primitives in recursive or conditional utilities.
- Brands cannot be used with `instanceof`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Gatekeeper factory pattern

```typescript
// example_gatekeeper.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

type UserId = Brand<string, "UserId">;

const UserId = {
  from: (id: string): UserId => {
    if (!id.startsWith("user_")) {
      throw new Error(`Invalid UserId: ${id}`);
    }
    return id as UserId;
  },
  generate: (): UserId => `user_${crypto.randomUUID()}` as UserId,
};

const uid = UserId.from("user_abc123");
console.log(uid); // Expected: user_abc123
```

**Expected Output:**
```
user_abc123
```

**Why this output occurs.** The `UserId` namespace object provides a validated `from` factory and a `generate` factory. All casts are confined to these functions, and callers cannot construct a `UserId` without going through validation.

---

#### Example 2: Branded IDs in a service boundary

```typescript
// example_service_ids.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;

function createUserId(id: string): UserId {
  if (!id.startsWith("user_")) throw new Error("Invalid UserId");
  return id as UserId;
}

function createOrderId(id: string): OrderId {
  if (!id.startsWith("order_")) throw new Error("Invalid OrderId");
  return id as OrderId;
}

function cancelOrder(orderId: OrderId, userId: UserId) {
  console.log(`Cancelling ${orderId} for ${userId}`);
}

const userId = createUserId("user_123");
const orderId = createOrderId("order_456");

cancelOrder(orderId, userId); // ✅ Correct order.
// cancelOrder(userId, orderId); // ❌ Error: swapped arguments.
```

**Expected Output:**
```
Cancelling order_456 for user_123
```

**Why this output occurs.** Branded types make the parameter order meaningful to the compiler. Swapping `UserId` and `OrderId` is a compile-time error, preventing a class of production bugs that structural typing would silently allow.

---

#### Example 3: Branded money in cents

```typescript
// example_brand_money.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

type Cents = Brand<number, "Cents">;

const Cents = {
  from: (n: number): Cents => {
    if (!Number.isInteger(n) || n < 0) throw new Error("Invalid cents");
    return n as Cents;
  },
  add: (a: Cents, b: Cents): Cents => (a + b) as Cents,
  format: (c: Cents): string => `$${(c / 100).toFixed(2)}`,
};

const price = Cents.from(1000);
const discount = Cents.from(100);

console.log(Cents.format(Cents.add(price, discount))); // Expected: $11.00
// Cents.from(10.5); // ❌ Throws: Invalid cents.
```

**Expected Output:**
```
$11.00
```

**Why this output occurs.** `Cents` is a branded number that only the `Cents.from` factory can create. Arithmetic is confined to `Cents.add`, ensuring integer arithmetic and preventing accidental use of floating-point dollars.

---

### Real-World Cases

- **E-commerce**: `UserId`, `OrderId`, `ProductId`, `Cents`, `CurrencyCode`.
- **Authentication**: `AccessToken`, `RefreshToken`, `HashedPassword`.
- **Content**: `SanitizedHTML`, `MarkdownString`, `Slug`.
- **Infrastructure**: `AbsolutePath`, `ProjectDir`, `TabId`.
- **Analytics**: `SessionId`, `EventName`, `AnonymousId`.

---

## 3. Opaque Types

### Definitions

**Core Definition.** **Opaque types**—a synonym for branded types—are types whose **underlying representation is hidden**, so consumers can use values of the type but cannot inspect or construct them without going through the module that defines them.

**Technical Definition.** In TypeScript, opaque types are typically implemented with the **branding pattern**: a base type intersected with a phantom marker. The term “opaque” emphasizes **encapsulation**: the type’s structure is not visible to consumers, and construction is controlled by the defining module. In Flow, opaque types are a language feature; in TypeScript, they are a **user-land pattern**. Opaque types may be **fully opaque** (no operations available except those exported) or **partially opaque** (assignable to the base type, as with branding).

**Beginner-Friendly Explanation.** An opaque type is **a type you can use but cannot look inside**. It hides its internal representation so that only the module that defines it can create or inspect values of that type. Branded types are a lightweight form of opaque types.

### Purposes

- **To hide implementation details** and enforce encapsulation across module boundaries.
- **To control construction** so that only validated values enter the type.
- **To prevent consumers from depending on the internal representation** of a type.
- **To provide a stable public API** that can change its representation without breaking consumers.
- **To model abstract data types** such as validated strings, money, and identifiers.

### Syntax Rules and Structure

**Branding-based opaque type:**

```typescript
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

// Only this module can create an Email.
export type Email = Brand<string, "Email">;

export function parseEmail(raw: string): Email {
  const trimmed = raw.trim().toLowerCase();
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(trimmed)) {
    throw new Error(`Invalid email: ${raw}`);
  }
  return trimmed as Email;
}
```

**Syntax rules.**

- The brand key and utility may be **module-private** to prevent external construction.
- Only **exported factory functions** can create values of the opaque type.
- The opaque type may be **assignable to its base type** (partial opacity) or fully hidden (stronger opacity).
- Opaque types are **erased** and have no runtime representation.

**Constraints and limitations.**

- TypeScript has **no native `opaque` keyword**; the pattern is emulated.
- Consumers can still forge values with `as`, unless lint rules or code review prevent it.
- Fully opaque types require careful API design to expose necessary operations.

### Annotated Complete Step by Step Code Examples

#### Example 1: Module-scoped opaque type

```typescript
// example_opaque_email.ts
// email.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

export type Email = Brand<string, "Email">;

export function parseEmail(raw: string): Email {
  const trimmed = raw.trim().toLowerCase();
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(trimmed)) {
    throw new Error(`Invalid email: ${raw}`);
  }
  return trimmed as Email;
}
```

```typescript
// app.ts
import { parseEmail, type Email } from "./example_opaque_email";

const email: Email = parseEmail("  Ada@Example.com  ");
console.log(email); // Expected: ada@example.com
```

**Expected Output:**
```
ada@example.com
```

**Why this output occurs.** `Email` is opaque outside the module: consumers cannot create one directly. `parseEmail` normalizes and validates the input, returning the branded `Email`. The brand ensures only validated emails flow into APIs that require `Email`.

---

#### Example 2: Opaque numeric type

```typescript
// example_opaque_positive.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

export type PositiveInt = Brand<number, "PositiveInt">;

export function parsePositiveInt(n: number): PositiveInt {
  if (!Number.isInteger(n) || n <= 0) {
    throw new Error(`Expected positive integer, got ${n}`);
  }
  return n as PositiveInt;
}

console.log(parsePositiveInt(42)); // Expected: 42
// parsePositiveInt(-1); // ❌ Throws.
```

**Expected Output:**
```
42
```

**Why this output occurs.** `PositiveInt` is opaque outside the module. The `parsePositiveInt` function is the only way to create one, ensuring the invariant `n > 0` holds for every `PositiveInt` value.

---

### Real-World Cases

- **Validated domain primitives**: `Email`, `UrlString`, `PositiveInt`, `NonEmptyString`.
- **Security types**: `SanitizedHTML`, `HashedPassword`, `EncryptedString`.
- **File system types**: `AbsolutePath`, `RelativePath`, `FileExtension`.
- **API tokens**: `AccessToken`, `RefreshToken`, `ApiKey`.

---

## 4. Type Branding Best Practices

### Definitions

**Core Definition.** **Type branding best practices** are guidelines for using branded types **safely, ergonomically, and performantly**, avoiding forgeable brands, unbranded construction, and type-inference explosions.

**Technical Definition.** Best practices center on **using `unique symbol` brand keys**, **centralizing construction in gatekeeper functions**, **confining `as` casts to those functions**, **branding only where semantic distinction matters**, **combining brands with runtime validation**, and **avoiding brands on every primitive**. The “gatekeeper pattern” centralizes creation and validation; the “three-part pattern” pairs the type with a validator and a factory. Brands must be **erased and phantom**—they should add no runtime property and no runtime overhead.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep branded types from becoming a mess. The big rules are: use a unique symbol for the brand key; create branded values only through validated factories; do not brand everything; and remember that a brand is only as trustworthy as the validation behind it.

### Purposes

- **To maximize type safety** by using unforgeable unique symbol keys.
- **To centralize validation** so every branded value is guaranteed valid.
- **To avoid type-inference performance issues** by branding selectively.
- **To keep the API ergonomic** with clear factory functions and type guards.
- **To prevent accidental circumvention** of brands through unchecked casts.
- **To integrate with runtime validators** for external data.

### Syntax Rules and Structure

**Recommended brand utility:**

```typescript
declare const __brand: unique symbol;

type Brand<T, B extends string> = T & { readonly [__brand]: B };
```

**Gatekeeper factory:**

```typescript
function createUserId(id: string): UserId {
  if (!id.startsWith("user_")) throw new Error("Invalid UserId");
  return id as UserId;
}
```

**Three-part pattern (type + validator + factory):**

```typescript
export type SavedTabId = Id & Brand<"SavedTabId">;
export const SavedTabId = type("string").as<SavedTabId>();
export const generateSavedTabId = (): SavedTabId => generateId() as SavedTabId;
```

**Best-practice rules.**

1. **Use `unique symbol` for the brand key.** String-keyed brands are forgeable and can collide across libraries. A single shared `__brand: unique symbol` is the recommended generic pattern.
2. **Centralize construction in gatekeeper functions.** All `as` casts should be confined to `createX` / `parseX` factories; application code should never cast to a branded type directly.
3. **Brand only where semantic distinction matters.** Do not brand every string or number. Display names, free-text comments, and throwaway values do not need brands.
4. **Combine branding with runtime validation.** A brand is a promise that the value satisfies an invariant; the factory must actually enforce that invariant.
5. **Prefer the three-part pattern for runtime-generated IDs**: type + validator + factory, with consistent naming.
6. **Avoid complex brands in performance-sensitive type utilities.** Intersections with primitives can trigger heavy inference; keep brands shallow.
7. **Use type predicates for validation-and-narrowing.** `value is Email` combines runtime checking with compile-time narrowing.
8. **Document and lint against unchecked casts.** Code review and lint rules should reject bare `as UserId` outside factories.

**Component breakdown (branding decision tree):**

```
Do two values share a base type but have different semantics?
  ├── Yes → Do they cross domain boundaries or function parameters?
  │        ├── Yes → Use a branded type with a unique symbol brand.
  │        │        ├── Is the value validated at runtime?
  │        │        │   ├── Yes → Use a gatekeeper factory and/or type predicate.
  │        │        │   └── No → Add validation in the factory.
  │        │        └── Brand only the semantically critical values.
  │        └── No → A naming convention may suffice.
  └── No → Do not brand.
```

**Constraints and limitations.**

- Brands do not enforce runtime safety by themselves; validation is separate.
- Brands are forgeable with `as`; discipline is required.
- Complex branded primitives can slow type checking in recursive utilities.
- Teams not bought into the pattern will undermine it with casts.

### Annotated Complete Step by Step Code Examples

#### Example 1: Unique symbol vs. string-key brand

```typescript
// example_best_unique_symbol.ts
// ❌ String-key brand: forgeable and collision-prone.
// type WeakUserId = string & { __brand: "UserId" };
// const forged = "anything" as WeakUserId;

// ✅ Unique symbol brand: strong nominal identity.
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

type UserId = Brand<string, "UserId">;

function createUserId(id: string): UserId {
  if (!id.startsWith("user_")) throw new Error("Invalid UserId");
  return id as UserId;
}

const uid = createUserId("user_123");
console.log(uid); // Expected: user_123
```

**Expected Output:**
```
user_123
```

**Why this output occurs.** The unique symbol key cannot be replicated or collided across modules, making the brand stronger than a string-keyed `__brand` property. Construction is confined to `createUserId`.

---

#### Example 2: Confining casts to gatekeeper functions

```typescript
// example_best_gatekeeper.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

type Email = Brand<string, "Email">;

// ✅ The only cast in the codebase for Email.
function parseEmail(raw: string): Email {
  const trimmed = raw.trim().toLowerCase();
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(trimmed)) {
    throw new Error(`Invalid email: ${raw}`);
  }
  return trimmed as Email;
}

function sendEmail(email: Email) {
  console.log(`Sending to ${email}`);
}

sendEmail(parseEmail("ada@example.com")); // Expected: Sending to ada@example.com
// sendEmail("raw@example.com"); // ❌ Error: string not assignable to Email.
```

**Expected Output:**
```
Sending to ada@example.com
```

**Why this output occurs.** The `as Email` cast exists only inside `parseEmail`, which performs validation first. Application code cannot construct an `Email` without going through the factory, centralizing the trust boundary.

---

#### Example 3: Type predicate for validation and narrowing

```typescript
// example_best_predicate.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

type ValidatedEmail = Brand<string, "ValidatedEmail">;

function isValidEmail(value: string): value is ValidatedEmail {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
}

function sendEmail(email: ValidatedEmail) {
  console.log(`Sending to ${email}`);
}

const input: string = "ada@example.com";
if (isValidEmail(input)) {
  sendEmail(input); // ✅ narrowed to ValidatedEmail.
}
```

**Expected Output:**
```
Sending to ada@example.com
```

**Why this output occurs.** The type predicate combines runtime validation with compile-time narrowing, eliminating the need for a separate cast.

---

#### Example 4: Avoiding brand overuse

```typescript
// example_best_selective.ts
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [__brand]: B };

// ✅ Brand semantic IDs.
type UserId = Brand<string, "UserId">;

// ❌ Do not brand display names or free-text.
// type DisplayName = Brand<string, "DisplayName">; // unnecessary noise.

function greet(name: string) {
  console.log(`Hello, ${name}`);
}

greet("Ada"); // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** `UserId` is branded because mixing it with `OrderId` is a real bug. A display name has no invariant and no semantic confusion risk, so branding it would add friction without value.

---

### Real-World Cases

- **Production e-commerce systems**: branded `UserId`, `OrderId`, `ProductId`, and `Cents` with gatekeeper factories.
- **Validation boundaries**: `parseEmail`, `parseUrl`, `parsePositiveInt` as the only entry points for branded values.
- **Libraries**: Effect’s `Brand` module, `ts-brand-types`, `@prosopo/ts-brand`, and `@vicin/phantom` provide branded-type helpers.
- **Code review**: rejecting bare `as UserId` casts and requiring factory construction.

---

## References

- Branded Types - Learning TypeScript - https://www.learningtypescript.com/articles/branded-types
- How to Implement Branded Types in TypeScript - OneUptime Blog - https://github.com/OneUptime/blog/blob/master/posts/2026-01-30-how-to-implement-branded-types-in-typescript/README.md
- Branded Types in Production: The Architecture of Domain Integrity - entwickler.de - https://entwickler.de/typescript/branded-types-in-production-the-architecture-of-domain-integrity
- Branded Types Reference - curiositech/windags-skills - https://github.com/curiositech/windags-skills/blob/main/skills/typescript-advanced-patterns/references/branded-types.md
- Professional TypeScript: Improve Type Safety with Branded Types - tsecurity.de - https://tsecurity.de/de/2539804/IT+Programmierung/Professional+TypeScript:+Improve+Type+Safety+with+Branded+Types/
- TypeScript Playground - Nominal Typing Example - https://www.typescriptlang.org/play#example/nominal-typing
- Brand Types to Make Domain Distinctions Compile-Checked - pproenca/dot-skills - https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/clean-code-ts-react/references/data-branded-types.md
- Branded ID Convention: Three-Part Pattern - EpicenterHQ/epicenter - https://github.com/EpicenterHQ/epicenter/blob/main/specs/20260312T180000-branded-id-convention.md
- Branded Types Skill - iaskshahram/skills - https://github.com/NeverSight/skills_feed/blob/main/data/skills-md/iaskshahram/skills/branded-types/SKILL.md
- @vicin/phantom - npm - https://www.npmjs.com/package/@vicin/phantom
- ts-brand-types - npm - https://www.npmjs.com/package/ts-brand-types
- @prosopo/ts-brand - npm - https://www.npmjs.com/package/@prosopo/ts-brand
- Opaque Types (Brand Types) - mcollina/skills - https://github.com/mcollina/skills/blob/main/skills/typescript-magician/rules/opaque-types.md
- Zod v4 Branded Primitive Types Performance Issue #4840 - GitHub - https://github.com/colinhacks/zod/issues/4840
- Anonymous Symbol Properties - microsoft/TypeScript Issue #64253 - https://github.com/microsoft/TypeScript/issues/64253
- Parse, Don’t Validate - Alexis King - https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/
- TypeScript Documentation - Type Compatibility (Structural Typing) - https://www.typescriptlang.org/docs/handbook/type-compatibility.html