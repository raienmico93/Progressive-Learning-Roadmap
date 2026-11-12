# Typing Async Functions in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Typing async functions** is the practice of explicitly and correctly annotating the types of `async` functions in TypeScript, whose return type is **always a `Promise<T>`** rather than `T`. Because `async` functions **implicitly wrap** their return values in promises, TypeScript's type system must model this wrapping precisely to preserve type safety across asynchronous boundaries, enable accurate inference, and prevent common errors such as awaiting non-promises or using unresolved values synchronously.

**Technical Definition.** An `async` function's declared return type must be `Promise<T>`, where `T` is the type of the value produced by the function's `return` statements (or `Promise<never>` if the function always throws). TypeScript **infers** `Promise<T>` when no annotation is present, but explicit annotations on public APIs improve error messages, stabilize contracts, and catch mismatches early. The return type participates in **covariance**: `Promise<Dog>` is assignable to `Promise<Animal>`. The `Awaited<T>` utility models the recursive unwrapping performed by `await`, and `ReturnType<typeof asyncFn>` yields `Promise<T>`, requiring `Awaited<ReturnType<...>>` to recover `T`.

**Beginner-Friendly Explanation.** Typing async functions means **writing the correct return type for a function that returns a promise**. When you mark a function as `async`, TypeScript always wraps its return value in a promise—so if the function returns a `User`, its type is `Promise<User>`, not `User`. Getting this right means callers know they must `await` the result, and TypeScript can catch mistakes where someone tries to use the value before the promise resolves.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **`Promise<T>` return type** | Async functions always return a promise, even when the body returns a plain value. |
| **Automatic wrapping** | `return value` inside an async function produces a resolved `Promise<T>`. |
| **Inference** | TypeScript infers `Promise<T>` from the returned expression when no annotation is present. |
| **Covariance** | `Promise<Dog>` is assignable to `Promise<Animal>`; narrower resolved types are compatible. |
| **`Promise<never>`** | Async functions that always throw have return type `Promise<never>`. |
| **`Awaited<T>`** | Models `await`'s recursive unwrapping at the type level. |
| **`ReturnType<typeof fn>`** | Yields `Promise<T>` for async functions; use `Awaited` to recover `T`. |
| **Rejection typing** | Rejection reasons are `any`; TypeScript does not enforce rejection type annotations. |

---

### Prerequisites

- **TypeScript basics**: functions, parameters, return types, and generics.
- **Promises**: `Promise<T>`, `.then()`, `.catch()`, and combinators.
- **Async/await syntax**: `async` functions and `await` expressions.
- **`Awaited<T>`** and `ReturnType<T>` utilities.
- **Structural typing** and covariance.
- **A TypeScript environment** with `target: "es2017"` or later (or downleveling).

---

### Related Programming Areas

- **Promises** — the foundation of async function return types.
- **Async/await** — the syntax that async function typing supports.
- **`Awaited<T>`** — the type-level unwrapping utility.
- **`ReturnType<T>`** — extracting async function return types.
- **Error handling** — `unknown` catch variables and rejection narrowing.
- **API design** — explicit async return types for public contracts.

---

## Core Concepts / Features

---

## 1. Typing Async Functions

### Definitions

**Core Definition.** **Typing async functions** is the practice of annotating an `async` function's return type as `Promise<T>`, where `T` is the type of the value produced by the function body.

**Technical Definition.** In TypeScript, an `async` function's return type is **always** `Promise<T>`. If the function body returns a value of type `T`, the function's type is `Promise<T>`. If the body returns a `Promise<T>`, the return type is also `Promise<T>` (the promise is **flattened**, not nested). If the function always throws, the return type is `Promise<never>`. Explicit annotations are recommended for exported and public functions; local functions may rely on inference.

**Beginner-Friendly Explanation.** Typing an async function means **writing `Promise<T>` instead of `T`**. If your function returns a `string`, its type is `Promise<string>`. TypeScript wraps the return value automatically, so callers always know they need to `await` it.

### Purposes

- **To make asynchronous return types explicit** for callers and tooling.
- **To prevent callers from using unresolved values** as if they were synchronous.
- **To catch return type mismatches** at compile time.
- **To stabilize public API contracts** across refactors.
- **To enable accurate inference** for consumers of the function.
- **To document the async nature** of a function in its signature.

### Syntax Rules and Structure

**General syntax:**

```typescript
async function functionName(params: Type): Promise<T> {
  return value; // T is wrapped in Promise<T>
}
```

**Component breakdown:**

- `async` — the keyword that makes the function return a promise.
- `: Promise<T>` — the return type annotation; `T` is the resolved value type.
- `return value` — the value returned by the function; TypeScript wraps it in a resolved promise.

**Syntax rules.**

- The return type must be `Promise<T>` (or a subtype thereof); `T` alone is an error.
- If the body returns `Promise<T>`, the function's return type is `Promise<T>`, not `Promise<Promise<T>>`.
- If the body returns `T`, the function's return type is `Promise<T>`.
- If the body always throws, the return type is `Promise<never>`.
- Explicit annotations are optional; TypeScript infers `Promise<T>` when omitted.

**Constraints and limitations.**

- Rejection reasons are typed `any`; TypeScript does not enforce rejection types.
- Async functions cannot be constructors.
- `Promise<never>` async functions always reject; callers must handle the rejection.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit `Promise<T>` annotation

```typescript
// example_typing_async_basic.ts
// Step 1: Declare an async function with an explicit Promise<T> return type.
async function fetchUser(id: string): Promise<{ id: string; name: string }> {
  // Step 2: Return a plain object; TypeScript wraps it in a promise.
  return { id, name: "Ada" };
}

// Step 3: Callers must await the result.
async function main() {
  const user = await fetchUser("u1");
  console.log(user.name); // Expected: Ada
}

main();
```

**Setup.** Compile with `npx tsc example_typing_async_basic.ts --strict --target es2017` and run `node example_typing_async_basic.js`.

**Expected Output:**
```
Ada
```

**Why this output occurs.** The function returns a plain object of type `{ id: string; name: string }`, which TypeScript wraps in `Promise<{ id: string; name: string }>`. `await` unwraps it back to the object type, so `user.name` is typed `string`.

---

#### Example 2: Returning a promise from an async function (flattening)

```typescript
// example_typing_async_flatten.ts
async function fetchValue(): Promise<number> {
  return 42;
}

// Returning a promise: the return type is Promise<number>, not Promise<Promise<number>>.
async function fetchValueAgain(): Promise<number> {
  return fetchValue(); // Flattened automatically.
}

async function main() {
  const value = await fetchValueAgain();
  console.log(value); // Expected: 42
}

main();
```

**Expected Output:**
```
42
```

**Why this output occurs.** TypeScript **flattens** nested promises in async function return types. `return fetchValue()` returns `Promise<number>`, and the async function's type remains `Promise<number>`, not `Promise<Promise<number>>`.

---

#### Example 3: `Promise<never>` for functions that always throw

```typescript
// example_typing_async_never.ts
async function fail(message: string): Promise<never> {
  throw new Error(message);
}

async function main() {
  try {
    await fail("Something went wrong");
  } catch (error: unknown) {
    if (error instanceof Error) {
      console.error(error.message); // Expected: Something went wrong
    }
  }
}

main();
```

**Expected Output:**
```
Something went wrong
```

**Why this output occurs.** The function always throws, so its return type is `Promise<never>`. The rejection is caught and narrowed with `instanceof Error`.

---

### Real-World Cases

- **API clients**: `async function fetchUser(id: string): Promise<User>`.
- **Database access**: `async function findById(id: string): Promise<Entity | null>`.
- **File system operations**: `async function readConfig(path: string): Promise<Config>`.
- **Service methods**: `async function processOrder(order: Order): Promise<OrderResult>`.

---

## 2. `Promise<T>` Return Type

### Definitions

**Core Definition.** The **`Promise<T>` return type** is the type of an async function whose resolved value is `T`.

**Technical Definition.** `Promise<T>` is a generic interface representing a promise that resolves to `T`. For async functions, the return type is **always** `Promise<T>`, never `T`. TypeScript **infers** `T` from the returned expression, and **flattens** nested promises. The `Awaited<T>` utility recursively unwraps `Promise<T>` to `T`, modeling `await`. The `ReturnType<typeof asyncFn>` utility yields `Promise<T>`, so `Awaited<ReturnType<typeof asyncFn>>` recovers `T`.

**Beginner-Friendly Explanation.** `Promise<T>` is **a promise that will eventually give you a `T`**. If your async function returns a `User`, its type is `Promise<User>`. When callers `await` it, they get the `User` back.

### Purposes

- **To type asynchronous return values precisely**.
- **To propagate resolved types** through promise chains and `async`/`await`.
- **To enable covariance** so narrower resolved types are assignable to broader ones.
- **To integrate with `Awaited<T>`** for type-level promise unwrapping.
- **To support `ReturnType<T>`** for extracting async function return types.

### Syntax Rules and Structure

**Return type annotation:**

```typescript
async function f(): Promise<T> { ... }
```

**Type extraction:**

```typescript
type FnReturn = ReturnType<typeof f>;       // Promise<T>
type FnValue = Awaited<ReturnType<typeof f>>; // T
```

**Component breakdown:**

- `Promise<T>` — the promise type with resolved value `T`.
- `ReturnType<typeof f>` — the function's return type (`Promise<T>`).
- `Awaited<Promise<T>>` — the resolved value type `T`.

**Syntax rules.**

- The return type is `Promise<T>`, not `T`.
- Nested promises are flattened: `Promise<Promise<T>>` becomes `Promise<T>`.
- `Awaited<T>` recursively unwraps nested promises.
- `Promise<T>` is covariant in `T`.

**Constraints and limitations.**

- Rejection reasons are not typed.
- `Promise<T>` does not carry cancellation information; use `AbortSignal` parameters.

### Annotated Complete Step by Step Code Examples

#### Example 1: `ReturnType` and `Awaited`

```typescript
// example_typing_return_awaited.ts
async function fetchUser(id: string): Promise<{ id: string; name: string }> {
  return { id, name: "Ada" };
}

type FetchUserReturn = ReturnType<typeof fetchUser>; // Promise<{ id: string; name: string }>
type User = Awaited<FetchUserReturn>;                // { id: string; name: string }

const user: User = { id: "u1", name: "Ada" };
console.log(user.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `ReturnType<typeof fetchUser>` yields `Promise<{ id: string; name: string }>`. `Awaited` unwraps it to the resolved object type.

---

#### Example 2: Covariance of `Promise<T>`

```typescript
// example_typing_covariance.ts
interface Animal { name: string }
interface Dog extends Animal { breed: string }

async function fetchDog(): Promise<Dog> {
  return { name: "Rex", breed: "Labrador" };
}

async function main() {
  const animal: Promise<Animal> = fetchDog(); // ✅ Covariant
  const resolved = await animal;
  console.log(resolved.name); // Expected: Rex
}

main();
```

**Expected Output:**
```
Rex
```

**Why this output occurs.** `Promise<Dog>` is assignable to `Promise<Animal>` because `Dog` is assignable to `Animal`. The resolved value still has the `name` property.

---

### Real-World Cases

- **API clients**: `Promise<ApiResponse<T>>` for typed responses.
- **Repository methods**: `Promise<User | null>` for lookups.
- **Async utilities**: `Promise.all` with typed tuple results.

---

## 3. Async Function Best Practices

### Definitions

**Core Definition.** **Async function best practices** are guidelines for declaring and typing async functions in ways that maximize **type safety, readability, and maintainability**, while avoiding common pitfalls such as missing return types, incorrect annotations, and unhandled rejections.

**Technical Definition.** Best practices center on **always annotating return types on public async functions**, **never annotating with `T` instead of `Promise<T>`**, **using `Promise<never>` for always-throwing functions**, **avoiding `return await` unless inside `try`**, **typing `catch` variables as `unknown`**, and **enabling lint rules** such as `@typescript-eslint/promise-function-async` and `@typescript-eslint/no-misused-promises`.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep async functions safe and readable. The big rules are: always write `Promise<T>` as the return type; do not write `T`; use `Promise<never>` for functions that always throw; and narrow `unknown` errors in `catch` blocks.

### Purposes

- **To make async APIs predictable** for callers.
- **To prevent incorrect return type annotations** that confuse callers.
- **To catch errors early** with explicit contracts.
- **To keep async code readable** and consistent.
- **To enable lint rules** that catch unhandled promises.

### Syntax Rules and Structure

**Best-practice rules:**

1. **Always annotate return types on public async functions.** Explicit `Promise<T>` annotations document the resolved type and stabilize the contract.
2. **Never annotate with `T` instead of `Promise<T>`.** Async functions always return promises.
3. **Use `Promise<never>` for functions that always throw.** This accurately models the function's behavior.
4. **Avoid `return await` unless inside `try`.** Outside `try`, `return promise` is equivalent and avoids an extra microtask.
5. **Type catch variables as `unknown`.** Since TypeScript 4.4, `strict` does this automatically; narrow with `instanceof Error`.
6. **Enable `@typescript-eslint/promise-function-async`** to enforce that functions returning promises are marked `async`.
7. **Enable `@typescript-eslint/no-misused-promises`** to catch promises passed to void-returning callbacks.
8. **Prefer `async`/`await` over `.then()` chains** for readability and error handling.

**Component breakdown (async function typing decision tree):**

```
Is the function async?
  ├── Yes → Does it always throw?
  │        ├── Yes → Use `Promise<never>`.
  │        └── No → Use `Promise<T>` where T is the resolved value type.
  └── No → Does it return a promise?
           ├── Yes → Consider marking it `async` (or annotate `Promise<T>`).
           └── No → Use `T`.
```

**Constraints and limitations.**

- TypeScript cannot enforce rejection typing; document error types in JSDoc if needed.
- Explicit annotations may drift from the implementation; the compiler checks compatibility, but keep them in sync.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit annotation for public APIs

```typescript
// example_best_public_api.ts
// ✅ Explicit Promise<T> annotation documents the resolved type.
export async function fetchUser(id: string): Promise<{ id: string; name: string }> {
  const response = await fetch(`/users/${id}`);
  return response.json();
}

// Consumers get accurate inference.
import { fetchUser } from "./example_best_public_api";
async function main() {
  const user = await fetchUser("u1");
  console.log(user.name); // user.name is string
}
```

**Expected behavior.** The explicit annotation makes the resolved type visible in editor tooltips and generated documentation.

**Why this is best practice.** Public APIs benefit from explicit return types because they document the contract and catch accidental changes to the return type.

---

#### Example 2: Avoiding `T` instead of `Promise<T>`

```typescript
// example_best_not_t.ts
// ❌ Error: Async function must return Promise<T>.
// async function bad(): number {
//   return 42;
// }

// ✅ Correct.
async function good(): Promise<number> {
  return 42;
}

good().then((n) => console.log(n)); // Expected: 42
```

**Expected Output:**
```
42
```

**Why this output occurs.** An async function's return type must be `Promise<T>`; annotating with `T` is a compile-time error.

---

#### Example 3: `Promise<never>` for always-throwing functions

```typescript
// example_best_never.ts
async function fail(message: string): Promise<never> {
  throw new Error(message);
}

fail("error").catch((error: unknown) => {
  if (error instanceof Error) console.error(error.message); // Expected: error
});
```

**Expected Output:**
```
error
```

**Why this output occurs.** `Promise<never>` accurately models a function that always rejects. Callers must handle the rejection, and the resolved type is `never`.

---

#### Example 4: `return await` only inside `try`

```typescript
// example_best_return_await.ts
async function fetchValue(): Promise<number> {
  return 42;
}

// ✅ Inside try: `return await` catches errors.
async function insideTry(): Promise<number> {
  try {
    return await fetchValue();
  } catch (error: unknown) {
    console.error("Caught:", error);
    return 0;
  }
}

// ✅ Outside try: `return` without await avoids an extra microtask.
async function outsideTry(): Promise<number> {
  return fetchValue();
}

insideTry().then(console.log);  // Expected: 42
outsideTry().then(console.log); // Expected: 42
```

**Expected Output:**
```
42
42
```

**Why this output occurs.** `return await` inside `try` is necessary to catch errors. Outside `try`, `return fetchValue()` is equivalent and slightly more efficient.

---

### Real-World Cases

- **API clients**: explicit `Promise<User>` return types for public methods.
- **Service layers**: `Promise<void>` for operations that do not return data.
- **Error handling**: `Promise<never>` for validation failures that always throw.
- **Lint enforcement**: `promise-function-async` and `no-misused-promises` in CI.

---

## References

- TypeScript 2.1 Release Notes — `async`/`await` Support — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html#asyncawait-support
- TypeScript 4.5 Release Notes — The `Awaited` Type and `Promise` Improvements — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html#the-awaited-type-and-promise-improvements
- TypeScript 4.4 Release Notes — Defaulting to the `unknown` Type in Catch Variables — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html#defaulting-to-the-unknown-type-in-catch-variables
- TypeScript Handbook — Utility Types (`Awaited<T>`, `ReturnType<T>`) — https://www.typescriptlang.org/docs/handbook/utility-types.html
- MDN — `async` Function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
- MDN — `await` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
- TypeScript ESLint — `promise-function-async` Rule — https://typescript-eslint.io/rules/promise-function-async/
- TypeScript ESLint — `no-misused-promises` Rule — https://typescript-eslint.io/rules/no-misused-promises/
- TypeScript ESLint — `no-floating-promises` Rule — https://typescript-eslint.io/rules/no-floating-promises/
- TypeScript ESLint — `await-thenable` Rule — https://typescript-eslint.io/rules/await-thenable/
- Total TypeScript — Typing Async Functions — https://www.totaltypescript.com/books/total-typescript-essentials
- MDN — Promise — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise