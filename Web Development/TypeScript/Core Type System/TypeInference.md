## Type Inference

**Topic Overview.** Type inference is TypeScript’s mechanism for **automatically determining types** when explicit annotations are absent. It operates across variable initializers, function return expressions, default parameter values, array literals, and **contextual positions** where an expected type influences the types of subexpressions . Inference is the default mode of TypeScript: every value has a type, even when you never write one. The system is **local and syntax-directed**, not whole-program; it uses available syntactic information to assign and relate types, then **erases** them during emission. Inference interacts with **widening** (broadening literal types to their base types) and **narrowing** (refining types through control-flow analysis) to provide safety with minimal annotation burden.

---

## 1. Type Inference

### Definitions

**Core Definition.** **Type inference** is the process by which the TypeScript compiler **derives types automatically** from program structure when explicit type annotations are not provided.

**Technical Definition.** Type inference in TypeScript is a **local, bidirectional** analysis. It infers types from variable initializers, return expressions, default parameter values, array element candidates (the **best common type** algorithm), and **contextual types** imposed by surrounding expressions. The inferred type is then used for assignability checking and control-flow narrowing. Inference is **erased** at compile time, like all type-level information.

**Beginner-Friendly Explanation.** Type inference means **TypeScript figures out the type for you** when it is obvious from the code. If you write `let count = 42`, TypeScript knows `count` is a `number` without you writing `: number`. You only need annotations when the inferred type is not what you want.

### Purposes

- **To reduce annotation verbosity** while preserving static type safety.
- **To make code easier to read and maintain** by letting types flow from values rather than from redundant declarations.
- **To enable contextual typing** in callbacks and higher-order functions, where parameter types are inferred from expected function types.
- **To support gradual typing**, so unannotated JavaScript participates in static checking through inference.

### Syntax Rules and Structure

Type inference has **no dedicated syntax**; it operates on existing constructs. The principal inference sites are:

```
let inferredVariable = expression;               // variable inference
function fn() { return expression; }              // return type inference
function fn(param = defaultValue) { ... }         // default parameter inference
const array = [expr1, expr2, expr3];             // best common type inference
const callback: (e: Event) => void = (e) => { }; // contextual inference
```

**Component breakdown:**

- `let inferredVariable = expression` — the compiler infers the variable’s type from the expression’s type.
- `return expression` — the function’s return type is inferred from the returned expression.
- `param = defaultValue` — the parameter’s type is inferred from the default value.
- `[expr1, expr2, ...]` — the array’s element type is the **best common type** of the candidate expressions.
- `(e) => { }` in a contextually typed position — the parameter `e` receives its type from the expected function type.

**Syntax rules.**

- Inference is **optional**; explicit annotations always override it.
- Inferred types are **widened** unless constrained by `const`, `as const`, or a contextual type.
- In **strict mode** (`--strict`), unannotated positions that cannot be inferred produce implicit `any` errors under `noImplicitAny`.

**Constraints and limitations.**

- Inference is **local**; it does not perform whole-program or cross-module inference.
- Inference can produce **wider types than intended** (e.g., `let status = "active"` infers `string`, not `"active"`).
- Over-reliance on inference can make types less obvious to human readers; complex expressions may benefit from annotations.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic variable and return inference

```typescript
// example_inference_basic.ts
// Step 1: Inferred from initializer.
let count = 42;                 // inferred: number
let message = "hello";          // inferred: string
let isReady = true;             // inferred: boolean

// Step 2: Inferred return type.
function add(a: number, b: number) {
  return a + b;                 // inferred return: number
}

// Step 3: Use inferred types.
console.log(count);             // Expected: 42
console.log(message);           // Expected: hello
console.log(isReady);           // Expected: true
console.log(add(2, 3));         // Expected: 5
```

**Expected Output:**
```
42
hello
true
5
```

**Why this output occurs.** The compiler infers `number`, `string`, and `boolean` from the initializers. The return type of `add` is inferred as `number` because `a + b` with two `number` operands produces `number`. No annotations are required for static checking to succeed.

#### Example 2: Default parameter inference

```typescript
// example_inference_default.ts
// Step 1: Default parameter determines type.
function greet(name = "Guest"): string {
  return `Hello, ${name}`;
}

// Step 2: Call with and without an argument.
console.log(greet());           // Expected: Hello, Guest
console.log(greet("Ada"));      // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Guest
Hello, Ada
```

**Why this output occurs.** The default value `"Guest"` is a string, so `name` is inferred as `string`. The function can be called with no argument (using the default) or with a string argument. The return type is explicitly annotated as `string`.

### Real-World Cases

- **Local variables**: `const total = items.reduce(...)` infers `number` without annotation.
- **Callbacks**: `[1, 2, 3].map(x => x * 2)` infers `x: number` and the result as `number[]`.
- **API clients**: Return types of small helper functions are inferred from their implementations, reducing boilerplate.

---

## 2. Inferred Types

### Definitions

**Core Definition.** **Inferred types** are the specific types that TypeScript **assigns** to declarations and expressions when no explicit annotation is present.

**Technical Definition.** An inferred type is the result of applying TypeScript’s inference algorithm at a given inference site. It may be a **widened** type (e.g., `string` for a `let` variable initialized with a string literal), a **literal type** (e.g., `"hello"` for a `const` variable), a **union** produced by best common type, or a type derived contextually. Inferred types are **not fixed**; they participate in control-flow narrowing and can be refined within branches.

**Beginner-Friendly Explanation.** Inferred types are **the types TypeScript decided on** after reading your code. If you write `const status = "active"`, the inferred type is `"active"` (a literal). If you write `let status = "active"`, the inferred type is `string` (widened). Inferred types are what you see when you hover over a variable in your editor.

### Purposes

- **To provide type safety without requiring annotations** on every declaration.
- **To make types visible through tooling**, so developers can understand the inferred type on hover.
- **To serve as the basis for widening and narrowing**, which refine inferred types over the course of execution.
- **To enable concise generic inference**, where type arguments are inferred from arguments and return values.

### Syntax Rules and Structure

Inferred types are observed, not written. The key syntactic determinants are:

```
const literalPreserved = "hello";   // inferred type: "hello" (literal)
let widened = "hello";              // inferred type: string (widened)
const arrayLiteral = [1, 2, 3];     // inferred type: number[]
const objectLiteral = { id: 1 };    // inferred type: { id: number }
```

**Component breakdown:**

- `const literalPreserved = "hello"` — `const` prevents widening; the inferred type is the literal `"hello"`.
- `let widened = "hello"` — `let` permits reassignment, so the literal is widened to `string`.
- `[1, 2, 3]` — the best common type of the elements is `number`, so the array type is `number[]`.
- `{ id: 1 }` — the object’s property type is widened to `number` unless `as const` is used.

**Syntax rules.**

- `const` declarations with literal initializers infer **literal types**; `let` and `var` infer **widened types**.
- Object and array literals are widened by default; `as const` preserves literals and makes properties `readonly`.
- Contextual types override default widening: a literal in a contextually typed position is inferred as the contextual literal type.

**Constraints and limitations.**

- Inferred types may be **too wide** for domain modeling; use literal unions or `as const` to narrow.
- Inferred types of exported values may expose implementation details; annotate public APIs for stability.
- Inferred types are not runtime values; they cannot be inspected at runtime.

### Annotated Complete Step by Step Code Examples

#### Example 1: `const` vs `let` inferred types

```typescript
// example_inferred_const_let.ts
// Step 1: const preserves literal type.
const literal = "hello";        // inferred: "hello"

// Step 2: let widens to string.
let widened = "hello";          // inferred: string

// Step 3: Reassignment behavior.
widened = "world";              // allowed (string)
// literal = "world";           // Error: Type '"world"' is not assignable to type '"hello"'.

console.log(literal);           // Expected: hello
console.log(widened);           // Expected: world
```

**Expected Output:**
```
hello
world
```

**Why this output occurs.** `const` prevents reassignment, so the literal `"hello"` is preserved as the inferred type. `let` allows reassignment, so the type is widened to `string` to permit any string value. The attempted reassignment of `literal` would be a compile-time error because `"world"` is not assignable to `"hello"`.

#### Example 2: Array and object inferred types

```typescript
// example_inferred_array_object.ts
// Step 1: Array infers best common type.
const numbers = [1, 2, 3];      // inferred: number[]

// Step 2: Object properties are widened.
const user = { id: 1, name: "Ada" }; // inferred: { id: number; name: string }

console.log(numbers);           // Expected: [1, 2, 3]
console.log(user.id);           // Expected: 1
```

**Expected Output:**
```
[1, 2, 3]
1
```

**Why this output occurs.** The array elements are all `number`, so the array type is `number[]`. The object properties are widened to `number` and `string` because object literals are mutable by default. If `as const` were applied, `user` would have readonly literal properties.

### Real-World Cases

- **Tooling**: Hovering over an inferred variable in VS Code displays its inferred type, helping developers understand the code.
- **Generic functions**: `identity(42)` infers `T = number` from the argument, avoiding explicit type arguments.
- **Configuration objects**: Inferred types of config objects are often widened; `as const` or `satisfies` preserves precise literal information.

---

## 3. Best Common Type

### Definitions

**Core Definition.** The **best common type** is the type selected by TypeScript when inferring the element type of an array or the type of a union-like construct from **multiple candidate expressions**, choosing a type that is compatible with all candidates.

**Technical Definition.** When inferring from several expressions, TypeScript calculates a “best common type” from the candidate types. The algorithm considers each candidate and picks the type that is compatible with all other candidates . If no single candidate is a supertype of all others, the result is a **union** of the candidate types . For an empty set of candidates, the best common type is the empty object type `{}`.

**Beginner-Friendly Explanation.** The best common type is **the most specific type that fits all the values**. If you have an array with numbers and `null`, TypeScript picks `number | null` because no single type covers both without a union. If all elements are numbers, it picks `number`.

### Purposes

- **To infer a single usable type for heterogeneous collections** without requiring explicit annotations.
- **To produce a union type when candidates share no common supertype**, preserving all possible value types.
- **To support array literal inference** and generic inference across multiple arguments.

### Syntax Rules and Structure

Best common type inference occurs in array literals and multi-expression contexts:

```
const array = [expr1, expr2, expr3];
```

**Component breakdown:**

- `expr1`, `expr2`, `expr3` — candidate expressions, each with a type.
- The algorithm finds a type compatible with all candidate types.
- If one candidate is a supertype of all others, that type is selected.
- If no supertype exists, the result is the union of all candidate types.

**Syntax rules.**

- Best common type considers **only the provided candidates**; it does not invent a supertype that is not among them.
- If a contextual type is present, the best common type may factor it in.
- `null` and `undefined` candidates are not widened before best common type selection in contexts with a contextual type.

**Constraints and limitations.**

- The algorithm may fail to find a common supertype when candidates share structure but no candidate is a supertype; the fallback is a union.
- An explicitly annotated array type overrides best common type inference.
- Empty arrays infer `never[]` in some configurations or `any[]` in non-strict settings.

### Annotated Complete Step by Step Code Examples

#### Example 1: Best common type with a common supertype

```typescript
// example_best_common.ts
// Step 1: Array with numbers and null.
const values = [0, 1, null];

// Step 2: Inferred type is (number | null)[] under strictNullChecks.
console.log(values.length);     // Expected: 3
console.log(values[0]);         // Expected: 0
console.log(values[2]);         // Expected: null
```

**Expected Output:**
```
3
0
null
```

**Why this output occurs.** The candidates are `number` (from `0` and `1`) and `null`. No single candidate is a supertype of all others, so the best common type algorithm produces the union `number | null`, and the array type is `(number | null)[]`.

#### Example 2: Union fallback when no common supertype exists

```typescript
// example_best_common_union.ts
class Rhino { horn = true; }
class Elephant { trunk = true; }
class Snake { venomous = true; }

const zoo = [new Rhino(), new Elephant(), new Snake()];
// Inferred type: (Rhino | Elephant | Snake)[]

console.log(zoo.length);        // Expected: 3
```

**Expected Output:**
```
3
```

**Why this output occurs.** None of the class instances is a supertype of the others, and there is no common `Animal` supertype among the candidates. The best common type algorithm falls back to the union `(Rhino | Elephant | Snake)[]` . An explicit annotation `const zoo: Animal[] = [...]` would produce `Animal[]` instead.

### Real-World Cases

- **Array literals**: `const mixed = [1, "two", true]` infers `(string | number | boolean)[]`.
- **Generic functions**: Multiple arguments may infer a union or a common supertype.
- **Test data**: Heterogeneous fixtures infer unions that preserve all possible value types.

---

## 4. Contextual Typing

### Definitions

**Core Definition.** **Contextual typing** is type inference that works **“in the other direction”**: the type of an expression is inferred from its **location** and the expected type imposed by its surrounding context.

**Technical Definition.** Contextual typing occurs when the type checker uses the expected type of an expression to infer the types of its subexpressions. For example, when a function expression is assigned to a variable with a known function type, the parameter types of the function expression are inferred from that expected type . Contextual typing is **bidirectional**: information flows from the context into the expression, supplementing the usual bottom-up inference.

**Beginner-Friendly Explanation.** Contextual typing means **the surrounding code tells TypeScript what type an inner expression should have**. If you assign a function to `window.onmousedown`, TypeScript knows the function’s parameter is a `MouseEvent` because that is the type `window.onmousedown` expects. You do not need to annotate the parameter yourself.

### Purposes

- **To reduce annotations in callbacks** by inferring parameter and return types from expected function types.
- **To make APIs easier to use** by allowing object literals and functions to inherit types from context.
- **To support bidirectional type checking**, where expected types propagate downward into expressions.

### Syntax Rules and Structure

Contextual typing occurs in **contextually typed positions**:

```
const handler: (event: MouseEvent) => void = (event) => {
  console.log(event.button);   // event contextually typed as MouseEvent
};
```

**Component breakdown:**

- `const handler: (event: MouseEvent) => void` — the **contextual type**.
- `(event) => { ... }` — the function expression; `event` has no annotation.
- The compiler infers `event: MouseEvent` from the contextual type.

**Syntax rules.**

- Contextual types flow into function parameters, object literal properties, array elements, and conditional expressions.
- An explicit annotation on the inner expression **overrides** the contextual type.
- Contextual typing works recursively: nested functions and objects inherit expected types from their context.

**Constraints and limitations.**

- Without a contextual type, unannotated function parameters may become `any` (under non-strict settings) or produce `noImplicitAny` errors.
- Contextual typing is **local**; it does not propagate across module boundaries or through complex indirection.
- Factoring a contextually typed expression into a standalone variable may lose the contextual type and require an annotation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Contextual typing in a DOM handler

```typescript
// example_contextual_dom.ts
// Step 1: Assign a function expression to a contextually typed position.
window.onmousedown = function (mouseEvent) {
  // Step 2: mouseEvent is contextually typed as MouseEvent.
  console.log(mouseEvent.button);   // Expected: 0 (for left click)
  // mouseEvent.kangaroo;            // Error: Property 'kangaroo' does not exist on type 'MouseEvent'.
};
```

**Expected Output (conceptual, when a mouse event occurs):**
```
0
```

**Why this output occurs.** `window.onmousedown` has a known type that expects a function receiving a `MouseEvent`. The function expression is contextually typed by that expected type, so `mouseEvent` is inferred as `MouseEvent`, which has a `button` property but not a `kangaroo` property .

#### Example 2: Contextual typing in an array callback

```typescript
// example_contextual_callback.ts
const numbers = [1, 2, 3];

// Step 1: The callback parameter is contextually typed.
const doubled = numbers.map((n) => n * 2);
// n is inferred as number from Array<number>.map's expected callback type.

console.log(doubled);   // Expected: [2, 4, 6]
```

**Expected Output:**
```
[2, 4, 6]
```

**Why this output occurs.** `numbers` is `number[]`, so `map` expects a callback of type `(value: number, index: number, array: number[]) => U`. The parameter `n` is contextually typed as `number`, allowing `n * 2` without an explicit annotation.

### Real-World Cases

- **React event handlers**: `onClick={(e) => e.preventDefault()}` infers `e` as `React.MouseEvent`.
- **DOM APIs**: Assigning handlers to `window` properties infers the event type from the property’s declared type.
- **Array methods**: `map`, `filter`, and `reduce` contextually type their callback parameters.

---

## 5. Type Widening

### Definitions

**Core Definition.** **Type widening** is the process by which TypeScript **broadens a literal type to its base type** — for example, inferring `string` rather than `"hello"` — typically when a value is assigned to a mutable location.

**Technical Definition.** Widening occurs when TypeScript infers the type of a `let` or `var` declaration, a mutable object property, or an array literal. Because the value may change, the compiler broadens literal types to their base types to permit reassignment . `const` declarations and `as const` assertions prevent widening, preserving literal types. Contextual types can also prevent widening when a literal is expected in a literal-typed position.

**Beginner-Friendly Explanation.** Type widening means **TypeScript makes a type less specific so it can change later**. If you write `let status = "active"`, TypeScript widens `"active"` to `string` because you might later assign `"inactive"`. If you write `const status = "active"`, it keeps the literal `"active"` because the value will never change.

### Purposes

- **To permit reassignment** of mutable variables to other values of the same base type.
- **To model mutable object properties and arrays** accurately, since their contents may change.
- **To prevent overly narrow types** from blocking legitimate reassignment.

### Syntax Rules and Structure

Widening is determined by the **declaration context**:

```
let widenedString = "hello";       // widened to string
const literalString = "hello";     // preserved as "hello"
const preserved = { status: "active" } as const; // readonly literal
```

**Component breakdown:**

- `let widenedString = "hello"` — `let` permits reassignment, so `"hello"` widens to `string`.
- `const literalString = "hello"` — `const` prevents reassignment, so the literal is preserved.
- `as const` — a **const assertion** that preserves literal types and makes properties `readonly` recursively .

**Syntax rules.**

- `let` and `var` declarations widen literal initializers to their base types.
- `const` declarations with literal initializers preserve literal types.
- Object and array literals widen their members unless `as const` is applied.
- Contextual literal types (e.g., a parameter typed `"up" | "down"`) prevent widening of literals in that position.

**Constraints and limitations.**

- Widening cannot be undone by reassignment; the variable’s declared type remains the widened type.
- `as const` makes values **deeply readonly**, which may be undesirable if mutation is needed.
- Widening can cause literal unions to lose precision; `as const` or explicit annotations preserve them.

### Annotated Complete Step by Step Code Examples

#### Example 1: `let` vs `const` widening

```typescript
// example_widening_let_const.ts
// Step 1: let widens literal.
let mutable = "active";          // inferred: string

// Step 2: const preserves literal.
const immutable = "active";      // inferred: "active"

mutable = "inactive";            // allowed (string)
// immutable = "inactive";       // Error: Type '"inactive"' is not assignable to type '"active"'.

console.log(mutable);            // Expected: inactive
console.log(immutable);          // Expected: active
```

**Expected Output:**
```
inactive
active
```

**Why this output occurs.** `let` permits reassignment, so the literal `"active"` is widened to `string`, allowing `"inactive"` to be assigned. `const` prevents reassignment, so the literal `"active"` is preserved, and assigning `"inactive"` is a compile-time error .

#### Example 2: `as const` preserving literals in objects

```typescript
// example_widening_as_const.ts
// Step 1: Without as const, properties widen.
const configWide = { status: "active" };
// inferred: { status: string }

// Step 2: With as const, literals are preserved and readonly.
const configNarrow = { status: "active" } as const;
// inferred: { readonly status: "active" }

console.log(configWide.status);   // Expected: active
console.log(configNarrow.status); // Expected: active
```

**Expected Output:**
```
active
active
```

**Why this output occurs.** The unasserted object literal widens `"active"` to `string` because object properties are mutable by default. `as const` preserves the literal type `"active"` and marks the property `readonly`, preventing widening .

### Real-World Cases

- **Configuration objects**: `as const` preserves literal keys and values for precise type derivation with `typeof`.
- **Action types**: Redux action type strings are preserved with `as const` to enable discriminated unions.
- **Route definitions**: `as const` preserves route path literals for type-safe routing.

---

## 6. Type Narrowing

### Definitions

**Core Definition.** **Type narrowing** is the process by which TypeScript **refines a broad type to a more specific type** within a branch of control flow, based on runtime checks such as `typeof`, `instanceof`, `in`, equality comparisons, truthiness checks, and user-defined type guards.

**Technical Definition.** Narrowing is the observable effect of **control-flow analysis**. TypeScript overlays static type analysis on JavaScript’s runtime control flow constructs — `if/else`, ternaries, loops, truthiness checks — and tracks the most specific possible type of a value at each program point . When a **type guard** (a runtime check that provides type information) is recognized, the compiler narrows the type within the guarded branch .

**Beginner-Friendly Explanation.** Type narrowing means **TypeScript gets more specific about a type after you check it**. If a variable is `string | number`, and you write `if (typeof value === "string")`, then inside that `if` block TypeScript knows `value` is a `string`. The check “narrows” the type from the union to one member.

### Purposes

- **To safely use union-typed values** by refining them to the specific type needed for an operation.
- **To enable strict null checks** by forcing `null` and `undefined` to be handled before use.
- **To support discriminated unions** and exhaustive state handling.
- **To leverage runtime checks** as compile-time type information through type guards.

### Syntax Rules and Structure

Narrowing is triggered by **type guards** in control-flow positions:

```
if (typeof value === "string") {
  value.toUpperCase();   // narrowed to string
}
```

**Component breakdown:**

- `typeof value === "string"` — a **`typeof` type guard**.
- Inside the `if` block, `value` is narrowed to `string`.
- Outside the block, `value` retains its original type (or the complementary narrowed type in the `else` branch).

Common narrowing constructs:

- `typeof x === "..."` — narrows primitives.
- `x instanceof ClassName` — narrows to a class instance.
- `"property" in x` — narrows by property presence.
- `x === literal` or `x !== literal` — narrows literal unions.
- `if (x)` — narrows out falsy values (under strict null checks).
- User-defined type guards: `function isT(x: unknown): x is T`.

**Syntax rules.**

- Narrowing occurs within the branch where the guard is true; the complementary type is available in the `else` branch.
- Discriminated unions are narrowed by checking a common literal-typed property.
- Type predicates (`x is T`) declare a user-defined guard’s narrowing effect.
- Assignments also narrow: assigning a specific type to a union variable narrows it to that type.

**Constraints and limitations.**

- Narrowing is **flow-sensitive** but not path-sensitive across function boundaries in all cases.
- `typeof null === "object"` is a JavaScript quirk; TypeScript accounts for it by not narrowing `null` out with `typeof x === "object"` alone .
- Type guards must be written carefully to avoid false positives; they do not perform deep runtime validation by default.

### Annotated Complete Step by Step Code Examples

#### Example 1: `typeof` narrowing

```typescript
// example_narrowing_typeof.ts
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;   // padding narrowed to number
  }
  return padding + input;                 // padding narrowed to string
}

console.log(padLeft(3, "hello"));         // Expected: "   hello"
console.log(padLeft(">>", "hello"));      // Expected: ">>hello"
```

**Expected Output:**
```
   hello
>>hello
```

**Why this output occurs.** The `typeof padding === "number"` guard narrows `padding` to `number` inside the `if` block, allowing `.repeat()`. After the `if`, the remaining type is `string`, allowing concatenation. The compiler verifies that both branches handle the narrowed type correctly .

#### Example 2: Discriminated union narrowing

```typescript
// example_narrowing_discriminated.ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(shape: Shape): number {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;   // narrowed to circle
  } else {
    return shape.side ** 2;               // narrowed to square
  }
}

console.log(area({ kind: "square", side: 2 })); // Expected: 4
```

**Expected Output:**
```
4
```

**Why this output occurs.** The discriminant property `kind` is a literal type in each union member. Checking `shape.kind === "circle"` narrows `shape` to the circle member inside the `if`; the `else` branch narrows to the square member. This enables safe access to `radius` and `side` respectively.

### Real-World Cases

- **Null handling**: `if (element !== null)` narrows `Element | null` to `Element` before calling DOM methods.
- **API responses**: Discriminated unions for success/error responses are narrowed by a `status` or `ok` field.
- **State machines**: Control-flow narrowing tracks the current state and restricts legal transitions.

---

## 7. Type Inference Best Practices

### Definitions

**Core Definition.** **Type inference best practices** are guidelines for **when to rely on inference and when to annotate explicitly**, aiming to maximize safety, readability, and maintainability while minimizing redundant annotations.

**Technical Definition.** Best practices center on the principle: **be explicit at boundaries, implicit within implementations** . Boundaries include function parameters, return types of public APIs, exported values, and component props. Implementations — local variables, internal expressions, callback parameters in contextually typed positions — benefit from inference. Additional practices include using `satisfies` to validate shapes without widening, `as const` to preserve literals when needed, and avoiding redundant annotations on trivially inferred types.

**Beginner-Friendly Explanation.** Best practices help you decide **when to write types and when to let TypeScript guess**. The golden rule: annotate things that are part of your code’s public surface (function inputs, outputs, exported interfaces), and let inference handle obvious local variables. Avoid annotating `const x: boolean = true` — TypeScript already knows. Use `satisfies` when you want to check a shape without losing precise literal types.

### Purposes

- **To maximize safety** by annotating the places where mistakes are most likely: boundaries and public APIs.
- **To reduce verbosity** by letting inference handle obvious local types and contextually typed callbacks.
- **To preserve precise literal types** when widening would lose valuable information, using `as const` or `satisfies`.
- **To align with official guidance** and avoid common inference pitfalls.

### Syntax Rules and Structure

**Key best-practice rules:**

1. **Be explicit at boundaries**: annotate function parameters and return types on exported functions .
2. **Be implicit within implementations**: omit annotations on local variables, callback parameters in contextually typed positions, and obvious expressions .
3. **Avoid redundant annotations**: `const x = 15` is preferred over `const x: number = 15` .
4. **Use `satisfies` to validate without widening**: it checks that an expression matches a type while preserving the expression’s inferred type .
5. **Use `as const` to preserve literals** when you need literal types, tuples, or readonly structures .
6. **Enable `--strict`**: strict null checks and `noImplicitAny` prevent inference from silently producing unsafe `any` or ignoring `null`/`undefined`.

**Syntax rules (boundary vs implementation):**

```
// Boundary: explicit parameter and return types.
export function calculateTotal(items: Item[]): number { ... }

// Implementation: inferred local types.
const total = items.reduce((sum, item) => sum + item.price, 0);
```

- `items: Item[]` — parameter annotation at a boundary.
- `): number` — return type annotation at a boundary.
- `const total = ...` — inferred local type; no annotation needed.

**Syntax rules (`satisfies`):**

```
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
} satisfies Record<string, string | [number, number, number]>;
```

- `satisfies` validates that `palette` conforms to the target type.
- The inferred type of `palette` is **preserved**, so `palette.green` remains `string` rather than widening to `string | [number, number, number]` .

**Constraints and limitations.**

- Over-annotating local variables can obscure the code and duplicate inference.
- `satisfies` is available only in TypeScript 4.9 and later.
- `as const` makes values deeply readonly, which may not be appropriate for mutable data.
- Inference cannot replace runtime validation at trust boundaries (e.g., network input).

### Annotated Complete Step by Step Code Examples

#### Example 1: Boundary annotation vs implementation inference

```typescript
// example_best_boundary.ts
interface Item {
  price: number;
}

// Boundary: explicit parameter and return types.
export function calculateTotal(items: Item[]): number {
  // Implementation: inferred local types.
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  return subtotal;
}

console.log(calculateTotal([{ price: 10 }, { price: 20 }]));
// Expected: 30
```

**Expected Output:**
```
30
```

**Why this output occurs.** The parameter and return types are explicitly annotated because this is an exported function — a boundary. The local `subtotal` is inferred as `number` from `reduce`, avoiding a redundant annotation. This keeps the public contract clear while minimizing internal noise .

#### Example 2: `satisfies` for shape validation without widening

```typescript
// example_best_satisfies.ts
type Colors = "red" | "green" | "blue";
type RGB = [red: number, green: number, blue: number];

// Use satisfies to validate the shape while preserving literals.
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  bleu: [0, 0, 255],   // Error: 'bleu' is not assignable to 'Colors'
} satisfies Record<Colors, string | RGB>;
```

**Expected Output (compile-time error):**
```
Property 'bleu' does not exist in type 'Record<Colors, string | RGB>'.
```

**Why this output occurs.** `satisfies` checks that `palette` conforms to the expected type, catching the `bleu` typo. At the same time, the inferred type of `palette` retains precise literal information for valid properties, so `palette.green` remains `string` rather than widening to `string | RGB` . This is the best of both worlds: validation plus precise inference.

#### Example 3: `as const` for literal preservation

```typescript
// example_best_as_const.ts
const routes = ["home", "about", "contact"] as const;
// inferred: readonly ["home", "about", "contact"]

type Route = typeof routes[number];
// Route = "home" | "about" | "contact"

const validRoute: Route = "about";
console.log(validRoute);   // Expected: about
```

**Expected Output:**
```
about
```

**Why this output occurs.** Without `as const`, `routes` would be inferred as `string[]`, and `typeof routes[number]` would be `string`, losing the literal union. `as const` preserves the tuple of literals and marks it readonly, enabling the precise `Route` union .

### Real-World Cases

- **Exported APIs**: Function parameters and return types are annotated; internal helpers rely on inference.
- **Configuration objects**: `satisfies` validates configuration shapes while preserving literal types for autocompletion.
- **React components**: Props are explicitly typed at boundaries; local state and callback parameters are often inferred.
- **Route tables**: `as const` preserves route literals for type-safe navigation.

---

## References

- TypeScript Handbook — Type Inference - https://www.typescriptlang.org/docs/handbook/type-inference.html 
- TypeScript Handbook — Narrowing - https://www.typescriptlang.org/docs/handbook/2/narrowing.html 
- TypeScript Playground — Type Widening and Narrowing - https://www.typescriptlang.org/play/typescript/language/type-widening-and-narrowing.ts.html 
- TypeScript 4.9 Release Notes — The `satisfies` Operator - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html#the-satisfies-operator 
- TypeScript Handbook — `const` Assertions (`as const`) - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- Effective TypeScript — Item 18: Avoid Cluttering Your Code with Inferable Types - https://github.com/danvk/effective-typescript
- Effective TypeScript — Item 24: Understand How Context Is Used in Type Inference - https://github.com/danvk/effective-typescript
- Steve Kinney — Type Inference (The Golden Rule) - https://stevekinney.com/courses/react-typescript/typescript-type-inference-mastery 
- TypeScript Wiki — FAQ (Contextual Typing) - https://github.com/microsoft/TypeScript/wiki/FAQ
- Google TypeScript Style Guide — Type Inference - https://google.github.io/styleguide/tsguide.html