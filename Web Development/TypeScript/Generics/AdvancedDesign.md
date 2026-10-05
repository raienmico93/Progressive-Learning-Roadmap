# TypeScript Advanced Generic Design: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Advanced generic design in TypeScript encompasses the techniques, patterns, and compiler mechanics that enable developers to build highly reusable, type-safe abstractions. It covers how the compiler deduces type arguments from usage, how to control or prevent that inference when needed, how to compose generic functions and types into complex frameworks, and how to extract and reconstruct generic parameters at the type level.

**Technical Definition**
Advanced generic design operates at the intersection of TypeScript's inference engine, conditional type system, and type-level programming capabilities. The compiler employs a multi-pass inference algorithm that collects constraints from arguments, contextual types, and return positions, then resolves type parameters by computing the best common candidate. The `NoInfer<T>` utility type (TypeScript 5.4+) allows developers to mark specific type parameter positions as non-inferring, preventing unwanted inference from secondary arguments. Higher-order generic functions propagate type parameters through function composition, while conditional types with `infer` enable type-level destructuring of generic structures. Generic composition combines these techniques to build framework-level abstractions where type safety is maintained across complex, multi-step pipelines.

**Beginner-Friendly Explanation**
Advanced generic design is about making generics work really well in complex situations. It's like being a master craftsperson with generics—you know not just how to write them, but how to control exactly what TypeScript infers, how to compose multiple generic functions together, and how to pull types apart at the type level. The newest tool is `NoInfer<T>`, which lets you tell TypeScript "don't guess the type from this argument—use the other one instead." Higher-order generic functions are functions that take generic functions and return generic functions, preserving type information through the chain. All of this enables building libraries and frameworks where the types just work, even for complex use cases.

### Key Characteristics

- **Multi-pass inference**: TypeScript uses multiple inference passes, prioritizing explicit annotations over contextual types and constraint candidates.
- **Context-sensitive deferral**: Contextually sensitive functions (those with unannotated parameters) are skipped during inference and revisited after other arguments are processed.
- **NoInfer control**: The `NoInfer<T>` utility type marks type parameter positions as non-inferring, excluding them from candidate collection.
- **Higher-order propagation**: Generic function type parameters propagate through higher-order functions under specific conditions.
- **Infer extraction**: Conditional types with `infer` enable type-level destructuring and parameter reconstruction.
- **Constraint validation**: `infer X extends Y` (TypeScript 4.7+) combines binding and constraint checking in one step.
- **Composition-first**: Advanced patterns favor composing small, focused generic functions over monolithic abstractions.

### Prerequisites

- Solid understanding of TypeScript generics fundamentals (type parameters, constraints, defaults)
- Familiarity with conditional types and the `infer` keyword
- Understanding of contextual typing and type inference mechanics
- Familiarity with utility types (`Parameters`, `ReturnType`, `InstanceType`)
- Experience with higher-order functions and functional programming concepts

### Related Programming Areas

- **Type Theory**: Constraint solving, unification, bounded quantification
- **Compiler Design**: Inference algorithms, multi-pass analysis, constraint prioritization
- **Functional Programming**: Function composition, currying, point-free style
- **Library Design**: Building reusable, type-safe APIs and frameworks
- **Type-Level Programming**: Conditional types, mapped types, template literal types

### Core Concepts / Features

1. Generic Inference Mechanics (How the Compiler Deduces Types from Arguments)
2. Controlling and Preventing Inference Using `NoInfer<T>`
3. Higher-Order Generic Functions and Nested Generic Resolution
4. Generic Composition and Reusable, Hyper-Type-Safe Framework Abstractions
5. Generic Parameter Reconstruction and Splitting (Extracting Generic Bounds via `infer`)


## 1. Generic Inference Mechanics (How the Compiler Deduces Types from Arguments)

### Definitions

**Core Definition**
Generic inference mechanics describe the algorithms and rules TypeScript's compiler uses to determine type arguments for generic functions and types based on the values and contexts at the call site. The compiler collects candidates from multiple positions and resolves them using a multi-pass, priority-ordered algorithm.

**Technical Definition**
TypeScript's generic inference algorithm operates in multiple passes. During a function call, the compiler processes arguments left-to-right, collecting inference candidates for each type parameter from argument positions, contextual types, and return positions. Contextually sensitive functions—those with parameters that lack explicit type annotations—are deferred to a later pass because their types may depend on the inferred type parameters. The compiler prioritizes explicit type annotations and direct argument matches over contextual types and return positions. When multiple candidates exist, TypeScript selects the best common type, falling back to constraints or defaults when no candidate can be inferred. Type parameter propagation through higher-order functions follows additional rules: the called function must be generic, return a function with a single call signature, and process arguments left-to-right without prior inferences for referenced type parameters.

**Beginner-Friendly Explanation**
When you call a generic function, TypeScript has to figure out what the type parameters should be. It does this by looking at the values you pass in. But it's smarter than just looking at one thing—it looks at all the arguments, the expected return type, and the context. If it sees a function without type annotations, it skips that initially and comes back later after it's figured out types from other arguments. This is why sometimes you can write code that "just works" even with complex generics. Understanding this helps you write code that TypeScript can infer correctly, and helps you debug when inference goes wrong.

### Purposes

- To understand why TypeScript infers certain types and not others.
- To write generic code that TypeScript can infer correctly without explicit annotations.
- To diagnose and fix inference failures in complex generic scenarios.
- To leverage multi-pass inference for better developer experience.
- To design APIs that work well with TypeScript's inference rules.

### Syntax Rules and Structure

**General Syntax: Inference from Arguments**

```typescript
function identity<T>(value: T): T {
  return value;
}

const result = identity(42);  // T inferred as number
```

**Component Breakdown**
- `T` is inferred from the argument `42`.
- The return type is `number` because `T` is `number`.

**General Syntax: Context-Sensitive Deferral**

```typescript
function callIt<T>(obj: {
  produce: (x: number) => T,
  consume: (y: T) => void,
}): void {
  // ...
}

callIt({
  consume: (y) => y.toFixed(),  // y is contextually sensitive — deferred
  produce: (x: number) => x * 2,  // x has explicit type — processed first
});
```

**Component Breakdown**
- `consume` is contextually sensitive (unannotated parameter).
- `produce` is processed first, inferring `T` as `number`.
- `consume` is then checked with `T = number`.

**General Syntax: Higher-Order Propagation**

```typescript
declare function pipe<A extends any[], B, C>(
  ab: (...args: A) => B,
  bc: (b: B) => C
): (...args: A) => C;

declare function list<T>(a: T): T[];
declare function box<V>(x: V): { value: V };

const listBox = pipe(list, box);
// <T>(a: T) => { value: T[] }
```

**Component Breakdown**
- `pipe` is generic over `A`, `B`, `C`.
- `list` and `box` are generic functions.
- The type parameters propagate through `pipe`, producing a generic result.

**Syntax Rules**

- Inference candidates are collected from argument positions, contextual types, and return positions.
- Contextually sensitive functions are deferred to a later pass.
- Arguments are processed left-to-right.
- Explicit type annotations take priority over inferred candidates.
- Higher-order propagation requires: the called function is generic, returns a function with a single call signature, and no prior inferences exist for referenced type parameters.
- Type parameter propagation follows left-to-right flow.

**Constraints and Limitations**

- Inference does not perform full unification; it's a practical algorithm optimized for developer experience.
- Higher-order propagation only works when types flow left-to-right.
- Context-sensitive deferral can fail in complex nested scenarios.
- Generic functions with multiple type parameters may infer wider types than desired.
- Inference may produce `unknown` when no candidate can be determined.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Pass Inference with Context-Sensitive Functions

```typescript
// Step 1: Define a generic function with a context-sensitive callback.
function callFunc<T>(callback: (x: T) => void, value: T): void {
  callback(value);
}

// Step 2: Call with a context-sensitive callback.
// TypeScript skips the callback during inference, infers T from `value`,
// then checks the callback with the inferred type.
callFunc((x) => x.toFixed(2), 42);
// x is number (inferred from value = 42)
// Output: "42.00" (if callback logs)

// Step 3: Demonstrate the multi-pass behavior.
function processItems<T>(
  items: T[],
  transform: (item: T) => string
): string[] {
  return items.map(transform);
}

const result = processItems([1, 2, 3], (n) => n.toFixed(2));
// n is number (inferred from items)
console.log(result);  // ["1.00", "2.00", "3.00"]

// Step 4: Higher-order propagation with pipe.
declare function pipe<A extends any[], B, C>(
  ab: (...args: A) => B,
  bc: (b: B) => C
): (...args: A) => C;

declare function list<T>(a: T): T[];
declare function box<V>(x: V): { value: V };

const listBox = pipe(list, box);
// listBox: <T>(a: T) => { value: T[] }
const boxList = pipe(box, list);
// boxList: <V>(x: V) => { value: V }[]

const x1 = listBox(42);
// x1: { value: number[] }
console.log(x1);  // { value: [42] }

const x2 = boxList("hello");
// x2: { value: string }[]
console.log(x2);  // [{ value: "hello" }]
```

**Expected Output:**
```
["1.00", "2.00", "3.00"]
{ value: [42] }
[{ value: "hello" }]
```

**Why This Output Occurs:** The `callFunc` function defers the context-sensitive callback, infers `T` from `value`, then checks the callback. The `pipe` function propagates type parameters from `list` and `box` through to the result. The `listBox` function remains generic, preserving the type parameter through the composition.

#### Example 2: Inference Failure and Recovery

```typescript
// Step 1: Define a generic function with a complex signature.
function createConfig<T extends Record<string, unknown>>(
  defaults: T,
  overrides: Partial<T>
): T {
  return { ...defaults, ...overrides } as T;
}

// Step 2: Call with simple arguments — inference works.
const config1 = createConfig(
  { host: "localhost", port: 3000 },
  { port: 8080 }
);
console.log(config1);  // { host: 'localhost', port: 8080 }

// Step 3: Complex call with context-sensitive objects — may need explicit types.
const config2 = createConfig(
  { host: "localhost", port: 3000, secure: false },
  { secure: true } as Partial<typeof config2Defaults>
);

// Step 4: Explicit type arguments resolve inference issues.
interface AppConfig {
  host: string;
  port: number;
  secure: boolean;
}

const config3 = createConfig<AppConfig>(
  { host: "localhost", port: 3000, secure: false },
  { port: 8080 }
);
console.log(config3);  // { host: 'localhost', port: 8080, secure: false }

// Step 5: When inference fails, TypeScript falls back to constraints.
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const value = getProperty({ name: "Alice", age: 30 }, "name");
// value: string (inferred from the object and key)
console.log(value);  // "Alice"
```

**Expected Output:**
```
{ host: 'localhost', port: 8080 }
{ host: 'localhost', port: 8080, secure: false }
Alice
```

**Why This Output Occurs:** The `createConfig` function infers `T` from `defaults` when possible. The explicit type argument `<AppConfig>` resolves ambiguity. The `getProperty` function infers `T` and `K` from the object and key, respecting the constraint `K extends keyof T`.

### Real-World Cases

**Case 1: React Custom Hooks**
Custom hooks like `useState<T>` and `useReducer<T>` rely on TypeScript's inference to determine state types from initial values.

**Case 2: API Client Methods**
Generic API client methods infer request and response types from the arguments and expected return type, reducing the need for explicit type arguments.

**Case 3: Testing Utilities**
Testing frameworks like Vitest and Jest use inference to type test bodies, assertions, and mock functions based on usage context.

**Case 4: Form Libraries**
Form libraries like React Hook Form and Formik use inference to derive form value types from initial values and validation schemas.


## 2. Controlling and Preventing Inference Using `NoInfer<T>`

### Definitions

**Core Definition**
`NoInfer<T>` is a utility type introduced in TypeScript 5.4 that marks a type parameter position as non-inferring. When a type is wrapped in `NoInfer<...>`, TypeScript does not use arguments at that position as candidates for type parameter inference, allowing developers to control which arguments drive inference.

**Technical Definition**
`NoInfer<T>` is an intrinsic utility type implemented as a special substitution type. When TypeScript performs type argument inference, it normally collects candidates from all positions where a type parameter appears. By wrapping a type parameter in `NoInfer<...>`, the position is excluded from candidate collection—TypeScript reads the type but does not use arguments at that position to infer the type parameter. This is particularly useful when a generic function has multiple parameters of the same type parameter, and only one should drive inference (typically the "source of truth" parameter). The `NoInfer` type resolves to `T` in all other contexts, so it does not change the final type.

**Beginner-Friendly Explanation**
`NoInfer<T>` lets you tell TypeScript "don't guess the type from this argument." Imagine you have a function with a list of allowed values and a default value. Without `NoInfer`, TypeScript might infer the type from the default value if it's not in the list, which is wrong. With `NoInfer`, you mark the default value's position so TypeScript only infers from the list. It's like saying "the list is the source of truth—the default just has to match it." This was introduced in TypeScript 5.4 and is especially useful for library authors who want precise control over their generic APIs.

### Purposes

- To prevent unwanted type inference from secondary arguments.
- To designate a specific parameter as the "source of truth" for inference.
- To produce better error messages when arguments don't match the intended type.
- To avoid the awkwardness of using a second type parameter just to constrain inference.
- To build more predictable generic APIs for library consumers.

### Syntax Rules and Structure

**General Syntax: `NoInfer<T>` in Function Parameters**

```typescript
function functionName<T>(
  source: T[],
  fallback: NoInfer<T>
): T {
  // T is inferred only from `source`
}
```

**Component Breakdown**
- `T[]`: The source parameter drives inference.
- `NoInfer<T>`: The fallback parameter is excluded from inference.
- `T`: The return type uses the inferred `T`.

**General Syntax: `NoInfer<T>` with Constraints**

```typescript
function pick<T extends string>(
  options: T[],
  fallback: NoInfer<T>
): T {
  return options[0] ?? fallback;
}
```

**Component Breakdown**
- `T extends string`: The constraint limits `T` to strings.
- `NoInfer<T>`: Only `options` drives inference.

**General Syntax: `NoInfer<T>` in Object Properties**

```typescript
function createFSM<TState extends string>(config: {
  initial: NoInfer<TState>;
  states: TState[];
}): TState {
  return config.initial;
}
```

**Component Breakdown**
- `states: TState[]`: The `states` array drives inference.
- `initial: NoInfer<TState>`: The `initial` property must match but does not drive inference.

**Syntax Rules**

- `NoInfer<T>` wraps a type parameter in any parameter position.
- The wrapped position is excluded from inference candidate collection.
- The type must still be assignable to the inferred type parameter.
- `NoInfer<T>` resolves to `T` in all other contexts.
- Available in TypeScript 5.4 and later.
- Can be applied to parameters, properties, and return types.
- Works with constraints and defaults.

**Constraints and Limitations**

- `NoInfer<T>` is only available in TypeScript 5.4+.
- It cannot prevent inference from all positions—at least one position must drive inference.
- If no other position can drive inference, `T` may fall back to `unknown` or its constraint.
- `NoInfer<T>` does not change the final type; it only affects inference candidate collection.
- Pre-5.4 codebases need a custom implementation (e.g., `type NoInfer<T> = [T][T extends any ? 0 : never]`).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Preventing Inference from Fallback Values

```typescript
// Step 1: Without NoInfer — inference from all positions.
function createSignalWithout<T>(initial: T, fallback: T): T {
  return initial ?? fallback;
}

const signal1 = createSignalWithout("active", "unknown");
// T inferred as "active" | "unknown" — too wide!

console.log(signal1);  // "active"

// Step 2: With NoInfer — inference only from the first parameter.
function createSignal<T>(initial: T, fallback: NoInfer<T>): T {
  return initial ?? fallback;
}

const signal2 = createSignal("active", "unknown");
// ❌ Error: Argument of type '"unknown"' is not assignable to parameter of type '"active"'.

// Step 3: Correct usage with matching values.
const signal3 = createSignal("active", "inactive");
// T inferred as "active" (from initial)
console.log(signal3);  // "active"

// Step 4: Array-based inference with NoInfer.
function pickOne<T extends string>(
  options: T[],
  fallback: NoInfer<T>
): T {
  return options[0] ?? fallback;
}

const picked = pickOne(["red", "green", "blue"], "red");
// T inferred as "red" | "green" | "blue"
console.log(picked);  // "red"

// pickOne(["red", "green"], "yellow");
// ❌ Error: "yellow" is not assignable to "red" | "green".
```

**Expected Output:**
```
active
active
red
```

**Why This Output Occurs:** Without `NoInfer`, `T` is inferred as the union of both arguments. With `NoInfer`, only the first argument drives inference, so `T` is `"active"`. The fallback must be assignable to `"active"`, which `"unknown"` is not. The `pickOne` function infers from `options` only, rejecting `"yellow"` because it's not in the list.

#### Example 2: `NoInfer` in FSM (Finite State Machine) Configuration

```typescript
// Step 1: Define an FSM factory with NoInfer.
function createFSM<TState extends string>(config: {
  initial: NoInfer<TState>;
  states: TState[];
}): TState {
  return config.initial;
}

// Step 2: Valid usage — initial is in the states list.
const fsm1 = createFSM({
  initial: "idle",
  states: ["idle", "loading", "success", "error"],
});
console.log(fsm1);  // "idle"

// Step 3: Invalid usage — initial is not in the states list.
// createFSM({
//   initial: "unknown",
//   states: ["idle", "loading", "success", "error"],
// });
// ❌ Error: Argument of type '"unknown"' is not assignable to parameter of type '"idle" | "loading" | "success" | "error"'.

// Step 4: NoInfer with object properties.
interface FormProps<T> {
  initialValues: T;
  onSubmit: (values: NoInfer<T>) => void;
}

function Form<T>({ initialValues, onSubmit }: FormProps<T>): void {
  // onSubmit does not drive T inference.
}

Form({
  initialValues: { name: "", email: "" },
  onSubmit: (values) => {
    // values: { name: string; email: string } (inferred from initialValues)
    console.log(values.name);
  },
});
```

**Expected Output:**
```
idle
```

**Why This Output Occurs:** The `createFSM` function uses `NoInfer<TState>` on the `initial` property, so only `states` drives inference. The `initial` property must match the inferred state type. In the `Form` component, `onSubmit` does not drive `T` inference—only `initialValues` does, so the callback receives the correct type.

### Real-World Cases

**Case 1: State Management Libraries**
State management libraries use `NoInfer` to ensure that reducer state types are inferred from the initial state, not from action payloads.

**Case 2: Form Libraries**
Form libraries use `NoInfer` to ensure form value types are inferred from initial values, not from submission handlers.

**Case 3: Router Libraries**
Routing libraries use `NoInfer` to ensure route parameter types are inferred from route definitions, not from navigation calls.

**Case 4: Validation Libraries**
Validation libraries use `NoInfer` to ensure schema types are inferred from the schema definition, not from validation data.

**Case 5: Component Libraries**
React component libraries use `NoInfer` to ensure prop types are inferred from defaults, not from consumer overrides.


## 3. Higher-Order Generic Functions and Nested Generic Resolution

### Definitions

**Core Definition**
Higher-order generic functions are functions that accept generic functions as arguments or return generic functions as results. When a higher-order function operates on a generic function, TypeScript can propagate the type parameters of the input function to the output function under specific conditions.

**Technical Definition**
TypeScript 3.4 introduced higher-order type inference from generic functions, implemented in PR #30215. When an argument expression in a function call is of a generic function type, the type parameters of that function type are propagated onto the result type of the call if: (1) the called function is a generic function that returns a function type with a single call signature, (2) that single call signature does not itself introduce type parameters, and (3) in the left-to-right processing of the function call arguments, no inferences have been made for any of the type parameters referenced in the contextual type for the argument expression. This enables functions like `pipe`, `compose`, and `flip` to preserve genericity through composition. The algorithm is not a complete unification algorithm—it only works when types flow from left to right.

**Beginner-Friendly Explanation**
Higher-order generic functions are functions that work with other generic functions. When you compose generic functions together—like piping `list` into `box`—TypeScript can preserve the generic type parameters through the composition. This means the resulting function is still generic, even though it's made of two generic functions. The rules are specific: the functions have to flow left-to-right, and the inner function has to be a simple generic function without its own type parameters on the returned signature. When it works, it's magical—you get full type safety through function composition without writing any manual type annotations.

### Purposes

- To compose generic functions while preserving type parameters.
- To build functional utilities like `pipe`, `compose`, and `flip`.
- To enable type-safe middleware and pipeline patterns.
- To support functional programming patterns in TypeScript.
- To propagate genericity through higher-order abstractions.

### Syntax Rules and Structure

**General Syntax: Higher-Order Generic Function**

```typescript
declare function pipe<A extends any[], B, C>(
  ab: (...args: A) => B,
  bc: (b: B) => C
): (...args: A) => C;
```

**Component Breakdown**
- `A extends any[]`: The argument tuple type.
- `B`: The intermediate type.
- `C`: The final type.
- Returns a function from `A` to `C`.

**General Syntax: Generic Function Composition**

```typescript
declare function list<T>(a: T): T[];
declare function box<V>(x: V): { value: V };

const listBox = pipe(list, box);
// <T>(a: T) => { value: T[] }
```

**Component Breakdown**
- `list` and `box` are generic functions.
- `pipe` propagates their type parameters.

**General Syntax: Flip Higher-Order Function**

```typescript
const flip = <A, B, C>(f: (a: A, b: B) => C) =>
  (b: B, a: A) => f(a, b);

const zip = <T, U>(x: T, y: U): [T, U] => [x, y];
const flipped = flip(zip);
// <T, U>(b: U, a: T) => [T, U]
```

**Component Breakdown**
- `flip` reverses the parameter order of a binary function.
- The type parameters propagate through `flip`.

**Syntax Rules**

- The called function must be generic and return a function type.
- The returned function type must have a single call signature.
- The call signature must not introduce its own type parameters.
- No inferences must have been made for referenced type parameters before processing the argument.
- Arguments are processed left-to-right.
- Type parameter propagation follows left-to-right flow.

**Constraints and Limitations**

- Only works when types flow left-to-right.
- Does not work with conditional types or overloads.
- Does not work when the arrow function lacks explicit type parameters.
- Contextually typed arrow functions infer `any` (not a generic type) unless explicitly parameterized.
- The algorithm is not a complete unification algorithm.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Higher-Order Type Inference with `pipe`

```typescript
// Step 1: Define higher-order generic functions.
declare function pipe<A extends any[], B, C>(
  ab: (...args: A) => B,
  bc: (b: B) => C
): (...args: A) => C;

declare function list<T>(a: T): T[];
declare function box<V>(x: V): { value: V };

// Step 2: Compose list and box.
const listBox = pipe(list, box);
// listBox: <T>(a: T) => { value: T[] }

const boxList = pipe(box, list);
// boxList: <V>(x: V) => { value: V }[]

// Step 3: Use the composed functions.
const x1 = listBox(42);
// x1: { value: number[] }
console.log(x1);  // { value: [42] }

const x2 = boxList("hello");
// x2: { value: string }[]
console.log(x2);  // [{ value: "hello" }]

// Step 4: Flip function with generic propagation.
const flip = <A, B, C>(f: (a: A, b: B) => C) =>
  (b: B, a: A) => f(a, b);

const zip = <T, U>(x: T, y: U): [T, U] => [x, y];
const flipped = flip(zip);
// <T, U>(b: U, a: T) => [T, U]

const t1 = flipped(10, "hello");
// [string, number]
console.log(t1);  // ["hello", 10]

const t2 = flipped(true, 0);
// [number, boolean]
console.log(t2);  // [0, true]
```

**Expected Output:**
```
{ value: [42] }
[{ value: "hello" }]
["hello", 10]
[0, true]
```

**Why This Output Occurs:** The `pipe` function propagates the type parameters of `list` and `box` through the composition, producing generic results. The `flip` function reverses the parameter order of `zip` while preserving the generic type parameters.

#### Example 2: Limitations of Higher-Order Inference

```typescript
// Step 1: Higher-order inference with explicit type parameters works.
const f1 = pipe(<U>(x: U) => [x], box);
// <U>(x: U) => { value: U[] }

// Step 2: Without explicit type parameters, inference falls back to any.
const f2 = pipe((x) => [x], box);
// (x: any) => { value: any[] } — NOT generic!

// Step 3: Workaround — annotate the arrow function explicitly.
const f3 = pipe(<T,>(x: T) => [x], box);
// <T>(x: T) => { value: T[] }

// Step 4: Conditional types and overloads do not support higher-order inference.
type WrongReturn<T> = T extends (...args: any[]) => infer R ? R : never;
// Higher-order inference does not work with conditional types.

// Step 5: A complete unification algorithm is not implemented.
declare function compose<A, B, C>(
  f: (a: A) => B,
  g: (b: B) => C
): (a: A) => C;

// compose(g, f) — order matters! Left-to-right only.
```

**Expected Output:** No runtime output (compile-time behavior only). The explicit-type-parameter version compiles with generic types; the unannotated version infers `any`.

**Why This Output Occurs:** Higher-order inference only works when the arrow function has explicit type parameters. Without them, TypeScript infers `any`. Conditional types and overloads do not support higher-order inference. The algorithm is left-to-right only.

### Real-World Cases

**Case 1: Functional Programming Libraries**
Libraries like fp-ts and Effect use higher-order generic functions for pipe, compose, and other functional utilities.

**Case 2: Middleware Composition**
Express-style middleware and Koa-style composition use higher-order generic functions to type middleware chains.

**Case 3: React Higher-Order Components**
React HOCs use higher-order generic functions to preserve component prop types through wrapping.

**Case 4: Redux Middleware**
Redux middleware uses higher-order generic functions to type store enhancers and middleware chains.

**Case 5: Testing Utilities**
Testing libraries use higher-order generic functions to compose test utilities while preserving types.


## 4. Generic Composition and Reusable, Hyper-Type-Safe Framework Abstractions

### Definitions

**Core Definition**
Generic composition is the practice of building complex, type-safe abstractions by combining smaller, focused generic functions and types. Hyper-type-safe framework abstractions are library-level designs that leverage generic composition to maintain complete type safety across multi-step pipelines, plugin systems, and configuration layers.

**Technical Definition**
Generic composition in framework design uses type parameters to thread type information through multiple layers of abstraction. Common patterns include: (1) generic factory functions that preserve type safety between internal steps without exposing internal types, (2) generic context interfaces for dependency injection that allow the same UI components to work with different state implementations, (3) generic message handler maps that compose type-safe handlers across modules, and (4) default generics patterns (`T = unknown`) that enable extensibility while maintaining reasonable defaults. These patterns combine higher-order inference, conditional types, and generic constraints to create abstractions where type safety is maintained end-to-end.

**Beginner-Friendly Explanation**
Generic composition is how you build big, type-safe things from small, type-safe pieces. Instead of writing one giant generic function that does everything, you compose several smaller ones. Framework abstractions take this further—they're the kind of design you see in libraries like tRPC, Zod, and TanStack Query. These libraries use generics to make sure that when you define a schema, the types flow through your entire application. The "default generics" pattern (`T = unknown`) is common: it lets libraries provide sensible defaults while still allowing users to specify their own types when needed.

### Purposes

- To build framework-level abstractions with end-to-end type safety.
- To compose smaller generic utilities into larger, more capable systems.
- To maintain type information across multiple layers of abstraction.
- To enable plugin and extension systems with type-safe integration.
- To provide sensible defaults while allowing full customization.

### Syntax Rules and Structure

**General Syntax: Generic Factory for DI Composition**

```typescript
function createClientFactory<TClient, TResult>(
  createClient: () => TClient,
  execute: (client: TClient) => TResult
): () => TResult {
  const client = createClient();
  return () => execute(client);
}
```

**Component Breakdown**
- `TClient`: The internal dependency type (not exposed).
- `TResult`: The result type (exposed).
- The factory preserves type safety between steps.

**General Syntax: Generic Context Interface**

```typescript
interface Context<TState, TActions, TMeta> {
  state: TState;
  actions: TActions;
  meta: TMeta;
}

function useProvider<TState, TActions, TMeta>(
  context: Context<TState, TActions, TMeta>
): void {
  // Same UI components work with different state implementations.
}
```

**Component Breakdown**
- `TState`, `TActions`, `TMeta`: Independent type parameters.
- The context is a contract any provider can implement.

**General Syntax: Default Generics Pattern**

```typescript
type Messages<R = unknown> = {
  [K in keyof R]: (data: R[K]) => void;
};
```

**Component Breakdown**
- `R = unknown`: Default enables simple usage.
- `R` can be specified for full type safety.

**Syntax Rules**

- Use type parameters to thread type information through composition.
- Expose only the types callers need (hide internal types).
- Use default generics (`T = unknown`) for extensibility.
- Compose small, focused generic functions rather than large monolithic ones.
- Use conditional types and inference to derive types from composition.
- Maintain left-to-right type flow for inference.

**Constraints and Limitations**

- TypeScript lacks native higher-kinded types, limiting some composition patterns.
- Deeply nested generic composition can produce complex error messages.
- Type inference may fail in complex composition scenarios.
- The `unknown` default can hide type errors if not carefully used.
- Framework abstractions require careful API design to remain usable.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Generic Factory for Type-Safe DI Composition

```typescript
// Step 1: Define a generic factory that composes DI steps.
type ClientFactory = {
  createClient: () => { request: (url: string) => Promise<unknown> };
  executeCall: (
    client: { request: (url: string) => Promise<unknown> },
    url: string
  ) => Promise<string>;
};

function createExecutor(factory: ClientFactory): (url: string) => Promise<string> {
  const client = factory.createClient();
  return (url: string) => factory.executeCall(client, url);
}

// Step 2: Implement the factory.
const executor = createExecutor({
  createClient: () => ({
    request: async (url: string) => `Response from ${url}`,
  }),
  executeCall: async (client, url) => {
    const result = await client.request(url);
    return String(result);
  },
});

// Step 3: Use the executor.
executor("/api/users").then((result) => console.log(result));
// "Response from /api/users"

// Step 4: Generic factory with type parameter for internal dependency.
function createTypedExecutor<TClient extends { request: (url: string) => Promise<unknown> }>(
  createClient: () => TClient,
  executeCall: (client: TClient, url: string) => Promise<string>
): (url: string) => Promise<string> {
  const client = createClient();
  return (url: string) => executeCall(client, url);
}

const typedExecutor = createTypedExecutor(
  () => ({ request: async (url: string) => `Typed: ${url}` }),
  async (client, url) => String(await client.request(url))
);

typedExecutor("/api/data").then(console.log);
// "Typed: /api/data"
```

**Expected Output:**
```
Response from /api/users
Typed: /api/data
```

**Why This Output Occurs:** The generic factory collapses the DI composition into a single function, using type parameters to preserve type safety between steps without exposing the internal client type to callers. The typed version uses a constraint to ensure the client has the required `request` method.

#### Example 2: Generic Context Interface for Framework Abstractions

```typescript
// Step 1: Define a generic context interface.
interface UIContext<TState, TActions, TMeta = unknown> {
  state: TState;
  actions: TActions;
  meta: TMeta;
}

// Step 2: Define a provider that creates the context.
function createUIContext<TState, TActions>(
  state: TState,
  actions: TActions
): UIContext<TState, TActions> {
  return { state, actions, meta: undefined };
}

// Step 3: Define components that consume the context.
function renderCounter(context: UIContext<{ count: number }, { increment: () => void }>): string {
  return `Count: ${context.state.count}`;
}

function renderToggle(context: UIContext<{ on: boolean }, { toggle: () => void }>): string {
  return `Toggle: ${context.state.on ? "ON" : "OFF"}`;
}

// Step 4: Create contexts with different state implementations.
const counterContext = createUIContext(
  { count: 0 },
  { increment: () => { /* ... */ } }
);

const toggleContext = createUIContext(
  { on: false },
  { toggle: () => { /* ... */ } }
);

console.log(renderCounter(counterContext));  // "Count: 0"
console.log(renderToggle(toggleContext));    // "Toggle: OFF"

// Step 5: The same UI components work with different state implementations.
console.log(renderCounter(createUIContext(
  { count: 42 },
  { increment: () => { /* ... */ } }
)));  // "Count: 42"
```

**Expected Output:**
```
Count: 0
Toggle: OFF
Count: 42
```

**Why This Output Occurs:** The generic `UIContext<TState, TActions, TMeta>` interface is a contract that any provider can implement. The same `renderCounter` and `renderToggle` functions work with different state implementations. The type parameters ensure type safety across different contexts.

### Real-World Cases

**Case 1: tRPC**
tRPC uses generic composition to propagate type information from server procedures to client calls, providing end-to-end type safety.

**Case 2: Zod**
Zod uses generic composition to derive TypeScript types from runtime schemas, ensuring that validation and types stay in sync.

**Case 3: TanStack Query**
TanStack Query uses default generics (`T = unknown`) and generic composition to type query keys, data, and error types.

**Case 4: Effect**
Effect uses advanced generic composition (including HKT emulation) to build type-safe effect systems.

**Case 5: React Hook Form**
React Hook Form uses generic composition to type form values, validation, and submission handlers.

**Case 6: SynthKernel**
SynthKernel is a type-safe, composable architecture for modular monoliths, combining OOP, advanced generics, and the Facade Pattern.

**Case 7: hyper-ts**
hyper-ts is an experimental middleware architecture that uses type-level information to enforce correct composition and abstraction for web servers.


## 5. Generic Parameter Reconstruction and Splitting (Extracting Generic Bounds via `infer`)

### Definitions

**Core Definition**
Generic parameter reconstruction is the type-level technique of extracting, splitting, and reconstructing generic type parameters using conditional types with the `infer` keyword. It enables type-level destructuring of generic structures, allowing developers to pull out inner types, reconstruct them with modifications, and build powerful type transformations.

**Technical Definition**
The `infer` keyword, used within conditional types (`T extends Pattern<infer U> ? ... : ...`), captures type information from within a type structure. When combined with template literal types, tuple types, and function signatures, `infer` enables extraction of array element types, promise values, function parameters, return types, and object property types. TypeScript 4.7 added `infer X extends Y`, which combines binding and constraint checking in a single step—the resulting type is exactly `Y`, not the wider position. Generic parameter reconstruction uses these extractions to rebuild types with modifications, such as reordering function parameters, transforming nested structures, or splitting unions based on extracted discriminants.

**Beginner-Friendly Explanation**
Generic parameter reconstruction is like taking apart a type and putting it back together differently. The `infer` keyword lets you say "if this type matches this pattern, capture the inner type as a variable." For example, if you have `Promise<string>`, you can use `infer` to extract `string`. You can do this for arrays, functions, objects, and even template literals. The advanced version, `infer X extends Y`, lets you both extract and constrain the type in one step—so you get exactly the type you need, not a wider version. This is how libraries build powerful type transformations that feel magical to use.

### Purposes

- To extract inner types from generic wrappers (arrays, promises, maps, etc.).
- To reconstruct types with modified structure.
- To split unions and extract discriminants.
- To validate extracted types with constraints (`infer X extends Y`).
- To build type-level parsing and transformation pipelines.

### Syntax Rules and Structure

**General Syntax: Basic `infer` Extraction**

```typescript
type ExtractArrayElement<T> = T extends (infer U)[] ? U : never;
```

**Component Breakdown**
- `T extends (infer U)[]`: Matches array types and captures the element type.
- `? U : never`: Returns the captured type or `never`.

**General Syntax: `infer` with Constraint (TypeScript 4.7+)**

```typescript
type ToNumber<S extends string> = S extends `${infer N extends number}` ? N : never;
```

**Component Breakdown**
- `` `${infer N extends number}` ``: Captures `N` and constrains it to `number`.
- The result is the number literal, not the string.

**General Syntax: Tuple Splitting with `infer`**

```typescript
type Split<T extends any[]> = T extends [infer First, ...infer Rest]
  ? { first: First; rest: Rest }
  : never;
```

**Component Breakdown**
- `[infer First, ...infer Rest]`: Splits a tuple into first element and rest.
- Returns an object with both parts.

**General Syntax: Function Parameter Extraction**

```typescript
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```

**Component Breakdown**
- `infer P`: Captures the parameter tuple.
- `infer R`: Captures the return type.

**Syntax Rules**

- `infer` can only be used within the true branch of a conditional type.
- `infer X` captures whatever type the position allows.
- `infer X extends Y` (TypeScript 4.7+) binds and constrains in one step.
- Multiple `infer` declarations can appear in a single conditional type.
- `infer` works with arrays, tuples, promises, functions, objects, and template literals.
- The extracted type can be used in the true branch and in subsequent conditionals.

**Constraints and Limitations**

- `infer` must be used with `extends` in a conditional type.
- `infer X extends Y` requires TypeScript 4.7+.
- Without a constraint, `infer X` captures the widest possible type.
- The constraint position cannot be a union with non-literal kinds (widens the result).
- Complex nested `infer` patterns can produce `unknown` or `never` unexpectedly.
- TypeScript lacks native higher-kinded types, limiting some reconstruction patterns.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `infer` Extraction Patterns

```typescript
// Step 1: Extract array element type.
type ArrayElement<T> = T extends (infer U)[] ? U : never;

type Test1 = ArrayElement<string[]>;       // string
type Test2 = ArrayElement<number[]>;       // number
type Test3 = ArrayElement<(string | number)[]>;  // string | number
type Test4 = ArrayElement<string>;         // never

// Step 2: Extract promise value.
type PromiseValue<T> = T extends Promise<infer U> ? U : never;

type Test5 = PromiseValue<Promise<string>>;  // string
type Test6 = PromiseValue<Promise<number>>;  // number

// Step 3: Extract object property type.
type GetData<T> = T extends { data: infer TData } ? TData : never;

type Test7 = GetData<{ data: string }>;      // string
type Test8 = GetData<{ data: number[] }>;    // number[]

// Step 4: Extract function return type.
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Test9 = MyReturnType<() => string>;     // string
type Test10 = MyReturnType<(a: number) => boolean>;  // boolean

console.log("Types extracted successfully.");
```

**Expected Output:**
```
Types extracted successfully.
```

**Why This Output Occurs:** Each conditional type uses `infer` to capture the inner type from a generic structure. The `ArrayElement` type extracts the element type from arrays, `PromiseValue` from promises, `GetData` from object properties, and `MyReturnType` from function signatures.

#### Example 2: Constrained `infer` and Parameter Reconstruction

```typescript
// Step 1: Constrained infer — extract number from template literal.
type ToNumber<S extends string> = S extends `${infer N extends number}` ? N : never;

type A = ToNumber<"42">;     // 42 (number literal)
type B = ToNumber<"forty">;  // never
type C = ToNumber<"3.14">;   // 3.14

console.log("Constrained infer works.");

// Step 2: Tuple splitting and reconstruction.
type Split<T extends any[]> = T extends [infer First, ...infer Rest]
  ? { first: First; rest: Rest }
  : never;

type SplitResult = Split<[string, number, boolean]>;
// { first: string; rest: [number, boolean] }

// Step 3: Function parameter reconstruction.
type ReverseArgs<T extends any[]> = T extends [infer First, ...infer Rest]
  ? [...ReverseArgs<Rest>, First]
  : [];

type Reversed = ReverseArgs<[string, number, boolean]>;
// [boolean, number, string]

// Step 4: Extract generic parameter from a complex type.
interface MyComplexInterface<TEvent, TContext, TPoint, TData> {
  event: TEvent;
  context: TContext;
  point: TPoint;
  data: TData;
}

type GetPoint<T> = T extends MyComplexInterface<any, any, infer TPoint, any>
  ? TPoint
  : never;

type Point = GetPoint<MyComplexInterface<string, number, boolean, object>>;
// boolean

// Step 5: Multiple infer slots.
type ExtractAll<T> = T extends MyComplexInterface<
  infer TEvent,
  infer TContext,
  infer TPoint,
  infer TData
> ? { event: TEvent; context: TContext; point: TPoint; data: TData }
  : never;

type All = ExtractAll<MyComplexInterface<string, number, boolean, object>>;
// { event: string; context: number; point: boolean; data: object }

console.log("Parameter reconstruction complete.");
```

**Expected Output:**
```
Constrained infer works.
Parameter reconstruction complete.
```

**Why This Output Occurs:** The `ToNumber` type uses `infer N extends number` to both extract and constrain the number. The `Split` type extracts the first element and rest of a tuple. The `ReverseArgs` type recursively reconstructs a tuple in reverse order. The `GetPoint` type extracts a specific generic parameter from a complex interface, and `ExtractAll` extracts all four generic parameters.

### Real-World Cases

**Case 1: Utility Type Libraries**
Libraries like `type-fest` and `ts-essentials` use `infer` extensively to build utility types like `PromiseValue`, `ArrayElement`, and `ReadonlyDeep`.

**Case 2: API Response Parsing**
API clients use `infer` to extract response types from generic API wrappers, enabling type-safe data access.

**Case 3: Form Validation**
Form libraries use `infer` to extract validation result types from schemas, ensuring type safety across validation and form state.

**Case 4: Router Libraries**
Routing libraries use `infer` to extract route parameters from path patterns, enabling type-safe parameter access.

**Case 5: State Machine Libraries**
State machine libraries use `infer` to extract state and event types from configuration objects, ensuring type-safe transitions.

**Case 6: Template Literal Parsing**
Libraries use `infer N extends number` to parse numeric values from template literals, enabling type-level arithmetic and validation.

**Case 7: Function Composition Libraries**
Functional programming libraries use `infer` to extract and reconstruct function parameter and return types, enabling type-safe composition.

---

## References

- TypeScript Handbook: Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook: Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript 5.4 Release Notes: `NoInfer` Utility Type — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-4.html#the-noinfer-utility-type
- TypeScript 6.0 Release Notes: Less Context-Sensitivity on `this`-less Functions — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-6-0.html
- TypeScript 3.4 Release Notes: Higher-Order Type Inference from Generic Functions — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#higher-order-type-inference-from-generic-functions
- TypeScript PR #30215: Higher Order Function Type Inference — https://github.com/microsoft/TypeScript/pull/30215
- TypeScript PR #56794: Add `NoInfer<T>` Intrinsic — https://github.com/microsoft/TypeScript/pull/56794
- TypeScript 4.7 Release Notes: `infer` `extends` Constraints — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html
- TypeScript Language Specification: Type Inference — https://github.com/microsoft/TypeScript/blob/main/doc/spec.md
- TypeScript Handbook: Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- Total TypeScript: NoInfer — TypeScript 5.4's New Utility Type — https://www.totaltypescript.com/noinfer-typescript-5-4-utility-type
- Total TypeScript: Use `infer` with Generics to Extract Types — https://www.totaltypescript.com/workshops/type-transformations/conditional-types-and-infer/extract-type-arguments-to-another-type-helper/solution
- Effective TypeScript: Item 14 — Use Type Operations and Generics to Avoid Repeating Yourself
- TypeScript Deep Dive: Generics — https://basarat.gitbook.io/typescript/type-system/generics
- Convex TypeScript Guide: NoInfer — https://www.convex.dev/typescript/advanced/type-operators-manipulation/noinfer
- Stack Overflow: Higher-Order Type Inference Limitations — https://stackoverflow.com/questions/79108367
- Stack Overflow: Extract Generic Parameter — https://stackoverflow.com/questions/50924506
- `ts-reset` Documentation — https://github.com/mattpocock/ts-reset
- Zod Documentation — https://zod.dev
- tRPC Documentation — https://trpc.io
- TanStack Query TypeScript Guide — https://tanstack.com/query/latest/docs/framework/react/typescript