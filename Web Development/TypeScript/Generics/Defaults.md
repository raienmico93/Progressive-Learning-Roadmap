# Default Type Parameters in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **default type parameter** is an optional type parameter in a generic declaration that is assigned a fallback type via the syntax `<T = DefaultType>`. When a type argument is neither explicitly supplied by the caller nor inferable from the surrounding context, the compiler substitutes the declared default type. Default type parameters were introduced in **TypeScript 2.3** and are governed by a precise set of rules concerning ordering, constraints, and inference fallback.

**Technical Definition.** In the type-parameter list of a generic function, interface, class, or type alias, a type parameter may carry a **default type** following an equals sign. A type parameter with a default is deemed **optional**; required type parameters must not follow optional ones. The default type must satisfy any constraint imposed on the parameter via `extends`. During resolution, defaults are applied **left to right**: a default may reference an earlier type parameter (whose resolved argument will be used) but a forward reference resolves to `{}`. When inference fails to choose a candidate, the default type is used if one is specified; otherwise, inference falls back to `{}`.

**Beginner-Friendly Explanation.** Think of a default type parameter as the **backup plan** for a generic type. If you call a generic function or use a generic type without saying what `T` should be, TypeScript uses the type you wrote after the equals sign. It is the type-level version of a **default function argument**: `function greet(name = "World")` uses `"World"` if you omit the argument; `<T = string>` uses `string` if you omit the type argument. Defaults make simple uses simple while still allowing full customization when needed.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Optionality** | A type parameter with a default is optional at call sites and in type references. |
| **Ordering constraint** | Required type parameters may not follow optional (defaulted) type parameters. |
| **Constraint compatibility** | The default type must be assignable to the parameter’s constraint, if one exists. |
| **Left-to-right resolution** | Defaults are resolved in declaration order; backward references use resolved arguments, forward references use `{}`. |
| **Inference fallback** | Defaults are used only when explicit type arguments are absent **and** inference cannot determine a candidate. |
| **Incremental adoption** | Defaults can be added to existing type parameters in merged interface/class declarations under documented conditions. |
| **Erased at runtime** | Like all TypeScript type constructs, defaults disappear entirely during compilation. |

---

### Prerequisites

- **Generics fundamentals**: declaring type parameters (`<T>`), constraints (`<T extends U>`), and type arguments (`<string>`).
- **Type inference**: how TypeScript infers type arguments from function arguments and contextual positions.
- **Basic TypeScript syntax**: functions, interfaces, type aliases, and classes.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compiler behavior.

---

### Related Programming Areas

- **Generic programming** in statically typed languages (Java, C#, Rust).
- **Default function parameters** in JavaScript and TypeScript — the value-level analogue.
- **Type inference and best-common-type algorithms**.
- **Type erasure** — defaults, like all type information, have no runtime representation.
- **Library API design** — defaults are a primary tool for making generic libraries ergonomic.

---

### Core Concepts / Features

The following three sections constitute the core of default type parameters.

---

## 1. Default Type Parameters

### Definitions

**Core Definition.** A **default type parameter** is a generic type parameter that carries an **optional fallback type**, declared with `=` in the type-parameter list, so that the parameter may be omitted by callers when a suitable type cannot or should not be specified.

**Technical Definition.** In a type-parameter list `<T, U = DefaultU>`, the parameter `U` is **optional**. The compiler maintains an ordered resolution process: when a type reference or call supplies fewer type arguments than the number of parameters, the remaining parameters are filled from their defaults, provided the defaults satisfy all constraints. If a default is absent and inference fails, the compiler infers `{}` for that parameter. The presence of a default also affects **inference**: if inference cannot choose a candidate for a defaulted parameter, the default type is selected rather than `{}`.

**Beginner-Friendly Explanation.** A default type parameter is **a generic slot with a pre-filled answer**. If you do not provide a type, and TypeScript cannot guess one from your code, it uses the pre-filled answer. This means you can write `Result<User>` instead of `Result<User, Error>` when the error type is usually `Error`.

### Purposes

- **To reduce verbosity at call sites** by allowing callers to omit type arguments that almost always take the same value.
- **To improve API ergonomics** in generic libraries, where simple uses should remain simple while advanced uses remain possible.
- **To provide a sensible fallback** when inference cannot determine a type candidate, avoiding the abrupt `{}` fallback.
- **To support gradual specification** of type arguments, letting callers specify only the leading type parameters they care about.
- **To enable safer library patterns** such as `T = unknown` defaults, which allow “any plugin” functions to accept specialized plugins without forcing every function to spell out unknown type arguments.

### Syntax Rules and Structure

**General syntax.**

```
TypeParameterList:
  < TypeParameter (, TypeParameter)* >

TypeParameter:
  BindingIdentifier Constraint? DefaultType?

Constraint:
  extends Type

DefaultType:
  = Type
```

**Component breakdown.**

```
<T, E = Error>
```

- `<` — opens the type-parameter list.
- `T` — a **required** type parameter (no default).
- `,` — separator.
- `E` — a type parameter name.
- `=` — **default-type separator**.
- `Error` — the **default type expression** (any valid type).
- `>` — closes the list.

**Syntax rules.**

1. **Ordering.** Required type parameters must not follow optional (defaulted) type parameters.
2. **Constraint satisfaction.** If a parameter has a constraint (`E extends Error`), its default type must satisfy that constraint (`E extends Error = Error` is valid; `E extends Error = string` is not).
3. **Left-to-right resolution.** Defaults are applied in order. A default may reference an **earlier** parameter; a reference to a **later** parameter resolves to `{}`.
4. **Partial specification.** When supplying type arguments, you may supply only the **required** parameters; unspecified optional parameters resolve to their defaults.
5. **Inference fallback.** If a default is specified and inference cannot choose a candidate, the default is inferred.
6. **Declaration merging.** A class or interface declaration that merges with an existing declaration may introduce a default for an existing type parameter, or introduce a **new** type parameter provided it has a default.

**Constraints and limitations.**

- A default does **not** add a constraint; it only supplies a fallback. `T = string` does not restrict `T` to `string` when an explicit argument is provided.
- Defaults are **erased** and have no runtime effect.
- Default type expressions cannot reference value-level constructs unless those constructs are valid in type positions (`typeof` works; arbitrary values do not).
- Adding a default to an existing parameter is a **non-breaking change** for most callers, but removing a default is breaking.

### Annotated Complete Step by Step Code Examples

#### Example 1: A `Result` type with a default error type

```typescript
// example_result_default.ts
// Step 1: Declare a generic type alias with two parameters.
// T is required; E defaults to Error.
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

// Step 2: Use only the required parameter — E resolves to Error.
const ok: Result<string> = { success: true, data: "hello" };
// Type: { success: true; data: string } | { success: false; error: Error }

// Step 3: Use both parameters with a custom error type.
const failed: Result<string, { code: number }> = {
  success: false,
  error: { code: 500 },
};

// Step 4: Log values.
console.log(ok.success);        // Expected: true
console.log(failed.success);    // Expected: false
```

**Setup.** Compile with `npx tsc example_result_default.ts --strict`. The default `E = Error` is used in the first declaration because no second type argument is supplied.

**Expected Output:**
```
true
false
```

**Why this output occurs.** `Result<string>` is equivalent to `Result<string, Error>` because `E` has a default. The compiler checks `ok` against the union with `error: Error`, and `failed` against the union with `error: { code: number }`. At runtime, the type information is erased; only the boolean assignments remain.

---

#### Example 2: Default referencing an earlier parameter

```typescript
// example_default_reference.ts
// Step 1: Declare a generic interface where the default for V
// references the earlier parameter K.
interface Dictionary<K extends string, V = K[]> {
  key: K;
  values: V;
}

// Step 2: Omit V — it defaults to K[].
const stringDict: Dictionary<"name"> = {
  key: "name",
  values: ["Alice", "Bob"], // V is "name"[] (a string array)
};

// Step 3: Supply V explicitly.
const numberDict: Dictionary<"age", number> = {
  key: "age",
  values: 30,
};

console.log(stringDict.values.length); // Expected: 2
console.log(numberDict.values);        // Expected: 30
```

**Expected Output:**
```
2
30
```

**Why this output occurs.** In `Dictionary<"name">`, `K` is resolved to `"name"`, and `V` defaults to `K[]`, which is `"name"[]`. Because `"name"` is a string literal, `"name"[]` is assignable to `string[]`, so the array of names is valid. In `Dictionary<"age", number>`, `V` is explicitly `number`, so `30` is valid. Defaults that refer to earlier parameters use the **resolved argument** of that earlier parameter, not its default.

---

#### Example 3: Required parameters cannot follow optional parameters

```typescript
// example_ordering.ts
// Step 1: This declaration is VALID — optional follows required.
type Valid<T, U = string> = { first: T; second: U };

// Step 2: This declaration is INVALID.
// type Invalid<T = string, U> = { first: T; second: U };
// Error: Required type parameters must not follow optional type parameters.

// Step 3: Use the valid type.
const value: Valid<number> = { first: 1, second: "hello" };
console.log(value.second); // Expected: hello
```

**Expected Output:**
```
hello
```

**Why this output occurs.** In `Valid<T, U = string>`, `T` is required and comes first, so the declaration is legal. `U` defaults to `string`. The invalid declaration `Invalid<T = string, U>` places a required parameter `U` after an optional one, which the compiler rejects because there would be no way to supply `U` without also supplying `T`.

---

### Real-World Cases

- **`Promise<T>` and `PromiseLike<T>`**: TypeScript’s own library definitions use default type parameters to make these interfaces easier to use without always specifying the resolved value type.
- **Result/Either types**: Libraries model success and failure with `Result<T, E = Error>`, so callers who only care about the success type can write `Result<User>`.
- **Event emitter maps**: `EventEmitter<Events extends Record<string, unknown[]> = Record<string, unknown[]>>` allows an untyped emitter to work out of the box while supporting precisely typed event maps when supplied.
- **React component props generics**: Generic components often default `T` to `unknown` or a broad object type so that simple usage requires no explicit type argument.

---

## 2. `<T = DefaultType>` Syntax

### Definitions

**Core Definition.** The syntax `<T = DefaultType>` is the concrete notation used to declare a **default type parameter**. It appears inside the angle-bracket type-parameter list of a generic declaration, with `T` as the parameter name and `DefaultType` as any valid TypeScript type expression.

**Technical Definition.** In the grammar of type-parameter declarations, the production is:

```
TypeParameter:
  BindingIdentifier Constraint? DefaultType?
DefaultType:
  `=` Type
```

The `=` token introduces the default type expression. The default is parsed as a **type**, not a value. The expression may be a primitive, a union, an intersection, a reference to another type parameter, a generic instantiation, a conditional type, a mapped type, or any other type-level construct.

**Beginner-Friendly Explanation.** `<T = DefaultType>` is the **exact thing you type** to give a generic a default. `T` is the placeholder, `=` means “if nothing is provided, use this,” and `DefaultType` is the fallback type. It is the type-level twin of `function f(x = 10)`.

### Purposes

- **To declare a fallback type concisely** without separate overloads or helper types.
- **To make generic APIs self-documenting** by showing the common case directly in the signature.
- **To allow the default to reference earlier type parameters**, enabling dependent defaults such as `V = K[]`.
- **To support constraint-aware defaults**, where the default itself is checked against an `extends` constraint.

### Syntax Rules and Structure

**General syntax (full form with constraint and default).**

```
<T extends Constraint = DefaultType>
```

**Component breakdown.**

```
<T extends Record<string, unknown> = Record<string, unknown>>
```

- `<` — opens the type-parameter list.
- `T` — the type parameter name.
- `extends Record<string, unknown>` — the **constraint** (`Constraint?`).
- `=` — the **default separator**.
- `Record<string, unknown>` — the **default type** (`DefaultType?`), which satisfies the constraint.
- `>` — closes the list.

**Syntax rules.**

- The default expression must be a **type**, not a value. `= 42` is invalid; `= 42` as a literal type is valid only if `42` is being interpreted as a type (which requires context).
- If a constraint is present, the default must satisfy it. The compiler checks this relationship statically.
- The default may reference an **earlier** type parameter: `<K, V = K[]>` is valid.
- The default may **not** reference a later type parameter in a meaningful way; forward references resolve to `{}` during default resolution.
- Multiple defaulted parameters may appear together, provided no required parameter follows them: `<A = string, B = number>` is valid; `<A = string, B>` is not.

**Constraints and limitations.**

- **No constraint implication.** A default does not constrain the parameter. `<T = string>` still permits `T` to be `number` when explicitly supplied.
- **Forward-reference limitation.** Defaults cannot depend on later type parameters; the compiler uses `{}` for forward references to avoid circularity checks.
- **`const enum` and value constraints.** Defaults are type-level; they cannot depend on runtime values except through type-level `typeof`.
- **Declaration merging strictness.** When merging declarations, defaults must be consistent; the compiler requires compatible default specifications.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic `<T = DefaultType>` syntax

```typescript
// example_syntax_basic.ts
// Step 1: Declare a generic type alias with a default.
type Box<T = string> = {
  value: T;
};

// Step 2: Omit the type argument — T defaults to string.
const stringBox: Box = { value: "hello" };

// Step 3: Supply a different type argument.
const numberBox: Box<number> = { value: 42 };

console.log(stringBox.value.toUpperCase()); // Expected: HELLO
console.log(numberBox.value.toFixed(2));    // Expected: 42.00
```

**Expected Output:**
```
HELLO
42.00
```

**Why this output occurs.** `Box` without an argument uses `T = string`, so `stringBox.value` is a `string` and `.toUpperCase()` is statically available. `Box<number>` overrides the default, so `numberBox.value` is a `number` and `.toFixed(2)` is available. The default is used only in the absence of an explicit type argument.

---

#### Example 2: Constraint and default together

```typescript
// example_syntax_constraint.ts
// Step 1: Constrain T and provide a default that satisfies the constraint.
type ErrorLike<T extends { message: string } = Error> = {
  error: T;
};

// Step 2: Omit T — it defaults to Error, which has a message property.
const defaultError: ErrorLike = { error: new Error("boom") };

// Step 3: Supply a custom error-like type.
const customError: ErrorLike<{ message: string; code: number }> = {
  error: { message: "failed", code: 500 },
};

console.log(defaultError.error.message); // Expected: boom
console.log(customError.error.code);     // Expected: 500
```

**Expected Output:**
```
boom
500
```

**Why this output occurs.** `Error` satisfies the constraint `{ message: string }`, so `T = Error` is a legal default. When `T` is omitted, the default `Error` is used, and `.message` is available. When a custom type with a `message` and `code` is supplied, it also satisfies the constraint, and `.code` is available on that branch.

---

#### Example 3: Partial specification of type arguments

```typescript
// example_syntax_partial.ts
// Step 1: Declare a generic with two parameters, the second defaulted.
type Pair<A, B = string> = {
  first: A;
  second: B;
};

// Step 2: Supply only the required first parameter.
const partial: Pair<number> = { first: 1, second: "one" };

// Step 3: Supply both parameters.
const full: Pair<number, boolean> = { first: 1, second: true };

console.log(partial.second.toUpperCase()); // Expected: ONE
console.log(full.second);                  // Expected: true
```

**Expected Output:**
```
ONE
true
```

**Why this output occurs.** In `Pair<number>`, `A` is resolved to `number`, and `B` is omitted, so it resolves to its default `string`. The compiler therefore checks `second` as a `string`. In `Pair<number, boolean>`, both arguments are supplied, overriding the default. Partial specification is legal because only required parameters must be supplied.

---

### Real-World Cases

- **`Record<K, V = unknown>`** in library code: the value type defaults to `unknown`, allowing `Record<"a" | "b">` without a second argument while preserving type safety.
- **`EventEmitter<Events = Record<string, unknown[]>>`**: an untyped emitter is immediately usable, and a typed emitter is available when an event map is supplied.
- **`QueryFn<TData = unknown, TContext = unknown>`** in data-fetching libraries: defaults let simple queries work without specifying every generic slot, while advanced queries can override each slot.

---

## 3. Default Type Parameter Best Practices

### Definitions

**Core Definition.** **Default type parameter best practices** are guidelines for deciding **when** to add a default, **what** default type to choose, and **how** to structure defaults so that APIs remain clear, safe, and genuinely more ergonomic rather than misleadingly optional.

**Technical Definition.** Best practices center on the principle that a default should represent a **meaningful “no-specialization” mode**. A default is appropriate when the generic type or function is **usefully instantiable** with the default values. Defaults should be avoided when the type is pointless without concrete type arguments, because a default then creates a **false sense of optionality** and weakens type safety by allowing overly broad types to propagate. For library authors, `unknown` is preferred over `any` as a default because it preserves safety and forces narrowing.

**Beginner-Friendly Explanation.** Use defaults **only when the default type actually makes sense for real use**. Do not add a default just to avoid writing a type argument. If a type only makes sense with specific types, then a default is lying to users by suggesting it can be used without them. When you do add a default, prefer `unknown` over `any` so the type stays safe.

### Purposes

- **To prevent misleading APIs** where defaults suggest flexibility that does not actually exist.
- **To maximize type safety** by avoiding defaults that silently widen types to `any` or overly broad unions.
- **To keep simple uses simple** without compromising advanced uses, especially in library design.
- **To document the common case** directly in the type signature so callers understand the intended usage.
- **To avoid accidental `any` propagation** by choosing `unknown` as the default for genuinely open slots.

### Syntax Rules and Structure

Best practices are **conventions**, not syntax, but they interact with the syntax rules. The key structural choices are:

1. **Default only after required parameters.**
2. **Default must satisfy the constraint.**
3. **Prefer `unknown` over `any`** for open-ended defaults.
4. **Consider whether the type is usable with the default** before adding one.
5. **Document why a default exists** so future maintainers do not remove it without understanding its purpose.

**Component breakdown (the “meaningful default” checklist):**

```
Does the type make sense with the default?
  ├── Yes → Is the default the most common specialization?
  │         ├── Yes → Add the default.
  │         └── No  → Consider no default or a different default.
  └── No  → Do NOT add a default; require explicit type arguments.
```

**Syntax rules for best-practice-compliant defaults.**

- **Rule 1 — Meaningful usability.** Add a default only if the type is useful with non-specific values.
- **Rule 2 — `unknown` over `any`.** For library APIs, default to `unknown` rather than `any` so callers must narrow before use.
- **Rule 3 — No “just in case” defaults.** If no one would ever intentionally use the default, remove it.
- **Rule 4 — Constraint-respecting defaults.** If a constraint exists, the default must satisfy it; do not weaken the constraint just to accommodate a default.
- **Rule 5 — Document library defaults.** Library authors should explain why defaults exist so consumers understand the intended usage modes.

**Constraints and limitations.**

- Defaults can **hide design problems**: a default that is never intentionally used is a code smell.
- `any` defaults **disable type safety** for the affected slot and propagate unsafely.
- Defaults in **merged declarations** must be consistent, which can complicate incremental API evolution.
- Defaults cannot fix **inference limitations**; they only provide a fallback when inference fails or is omitted.

### Annotated Complete Step by Step Code Examples

#### Example 1: Appropriate default — `unknown` for an open plugin slot

```typescript
// example_best_meaningful.ts
// Step 1: Define a plugin type with meaningful unknown defaults.
type Plugin<TConfig = unknown, TMethods = unknown> = {
  id: string;
  config: TConfig;
  methods: TMethods;
};

// Step 2: A function accepts ANY plugin without spelling out unknowns.
function logPlugin(plugin: Plugin): void {
  console.log(plugin.id);
}

// Step 3: Specialized plugins are accepted because they are structurally
//       compatible with Plugin<unknown, unknown>.
const analyticsPlugin: Plugin<{ apiKey: string }, { track: (e: string) => void }> = {
  id: "analytics",
  config: { apiKey: "secret" },
  methods: { track: (e) => console.log(e) },
};

logPlugin(analyticsPlugin); // Expected: analytics
```

**Expected Output:**
```
analytics
```

**Why this output occurs.** The defaults `TConfig = unknown` and `TMethods = unknown` make `Plugin` usable as a **minimal constraint**. `logPlugin` accepts `Plugin` without type arguments, and the specialized `analyticsPlugin` is assignable because its `config` and `methods` are compatible with `unknown`. Using `any` instead would also compile, but `unknown` preserves the requirement to narrow before use inside `logPlugin` if it ever accessed `config` or `methods`.

---

#### Example 2: Inappropriate default — a builder that requires specific values

```typescript
// example_best_inappropriate.ts
// ❌ WRONG: Defaults make this type look optional when it is not useful
// without concrete values.
interface ColumnBuilderState<
  Name extends string = string,
  Nullable extends boolean = boolean,
  Type extends string = string,
> {
  readonly name: Name;
  readonly nullable: Nullable;
  readonly type: Type;
}

// ✅ CORRECT: Remove defaults because the type is useless without specifics.
interface StrictColumnBuilderState<
  Name extends string,
  Nullable extends boolean,
  Type extends string,
> {
  readonly name: Name;
  readonly nullable: Nullable;
  readonly type: Type;
}

// The strict version forces callers to think about each dimension.
const column: StrictColumnBuilderState<"age", false, "number"> = {
  name: "age",
  nullable: false,
  type: "number",
};

console.log(column.name); // Expected: age
```

**Expected Output:**
```
age
```

**Why this output occurs.** In the first interface, the defaults `string`, `boolean`, and `string` create a type that **compiles but is meaningless**: a column with name `string` and type `string` does not describe a real column. The defaults suggest optionality where none exists. The strict version forces explicit specification, making the API honest and preventing accidental use of broad, useless types.

---

#### Example 3: Constraint-aware default with a type helper

```typescript
// example_best_constraint.ts
// Step 1: A type helper where the error slot defaults to undefined.
type CreateDataShape<TData, TError = undefined> = {
  data: TData;
  error: TError;
};

// Step 2: Omit TError — error is undefined.
const simple: CreateDataShape<string> = {
  data: "hello",
  error: undefined,
};

// Step 3: Supply an Error type.
const withError: CreateDataShape<string, Error> = {
  data: "hello",
  error: new Error("failed"),
};

console.log(simple.error);        // Expected: undefined
console.log(withError.error?.message); // Expected: failed
```

**Expected Output:**
```
undefined
failed
```

**Why this output occurs.** `TError = undefined` is a meaningful default: many data shapes have no error, and `undefined` accurately models that absence. When an error is possible, the caller supplies `Error`. If a constraint were added (`TError extends Error | undefined = undefined`), the default would still satisfy it. This demonstrates the principle that a default should represent a **real “no-specialization” mode**, not an arbitrary placeholder.

---

### Real-World Cases

- **TanStack Query, Zod, and tRPC** use `T = unknown` defaults in generic types so that simple usage requires no type arguments while advanced usage remains fully typed.
- **Prisma’s internal style guide** explicitly prohibits defaults on builder-state types unless the type is meaningfully usable with non-specific values.
- **Library plugin systems** default configuration and method slots to `unknown`, enabling `definePlugin({ id, config, methods })` without forcing every caller to specify unknown type arguments.
- **Type helpers** such as `CreateDataShape<TData, TError = undefined>` use defaults to model optional error channels without requiring every caller to write `undefined`.

---

## References

- TypeScript 2.3 Release Notes — Generic Parameter Defaults - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-3.html#generic-parameter-defaults
- Microsoft/TypeScript Pull Request #13487 — Adds Support for Type Parameter Defaults - https://github.com/microsoft/TypeScript/pull/13487
- Prisma Next — TypeScript Patterns: Generic Parameter Defaults - https://github.com/prisma/prisma-next/blob/e0e739ca6a0e076c97733ef30ec3bf7b1f43a27b/docs/reference/typescript-patterns.md
- Epicenter — The Default Generics Pattern: Why Libraries Use `T = unknown` - https://github.com/EpicenterHQ/epicenter/blob/main/docs/articles/typescript-default-generics.md
- Total TypeScript — Default Type Parameters in Generics - https://www.totaltypescript.com/workshops/typescript-pro-essentials/designing-your-types/default-type-parameters-in-generics
- Total TypeScript — Set a Default Type Value in a Type Helper - https://www.totaltypescript.com/workshops/type-transformations/type-helpers/optional-type-parameters-in-type-helpers/solution
- TypeScript Handbook — Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html