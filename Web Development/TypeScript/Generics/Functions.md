## Generic Functions

**Topic Overview.** A **generic function** is a function that declares one or more **type parameters** — type-level variables that stand in for concrete types supplied at the call site. Generic functions are TypeScript’s primary tool for building reusable, type-safe utilities that work over many types without sacrificing the type relationship between inputs and outputs. Unlike using `any`, generics **preserve type information**: the compiler knows that `identity(42)` returns a `number`, not `any`, and it can enforce that the argument and return types are consistent. Generic functions support **type parameter constraints** (`extends`), **defaults**, and **multiple type parameters**, and they rely heavily on **type argument inference** so that callers rarely need to write explicit type arguments. The TypeScript Handbook describes generics as the mechanism for creating “a component that can work over a variety of types rather than a single one,” allowing consumers to “use their own types”.

---

## 1. Generic Function Declaration

### Definitions

**Core Definition.** A **generic function declaration** is a function declaration that includes a **type parameter list** in angle brackets immediately after the function name (or before the parameter list for arrow functions), introducing one or more type variables used in the function’s signature and body.

**Technical Definition.** In TypeScript, a generic function declaration is written as `function name<T>(params): ReturnType { ... }`. The type parameters are declared before the function’s parentheses and are in scope throughout the parameter list, return type, and function body. Arrow functions place the type parameters before the parameter list: `const fn = <T>(arg: T): T => arg`. A function is generic if and only if it declares at least one type parameter.

**Beginner-Friendly Explanation.** A generic function is **a function with a placeholder type**. You write `<T>` after the function name to say “this function works with some type, and I’ll call it `T` for now.” Then you use `T` wherever you want to refer to that type — in parameters, return types, and inside the function. When you call the function, TypeScript replaces `T` with the actual type you use.

### Purposes

- **To create reusable functions** that operate over many types without duplicating logic or losing type safety.
- **To capture and preserve type relationships** between inputs and outputs, so the return type reflects the argument type.
- **To avoid the type-erasing behavior of `any`**, which accepts any value but discards all type information.
- **To enable type argument inference**, reducing the annotation burden on callers.

### Syntax Rules and Structure

**General syntax (function declaration):**

```typescript
function functionName<TypeParam>(param: TypeParam): TypeParam {
  return param;
}
```

**Component breakdown:**

- `functionName` — the function identifier.
- `<TypeParam>` — the **type parameter list**; declares the type variable(s).
- `param: TypeParam` — a parameter typed by the type parameter.
- `): TypeParam` — the return type, preserving the type relationship.

**General syntax (arrow function):**

```typescript
const functionName = <TypeParam>(param: TypeParam): TypeParam => param;
```

**Syntax rules.**

- Type parameters are declared in angle brackets **before the parameter list** for function declarations and **before the parameter list** for arrow functions.
- Type parameters may be used in parameter types, the return type, local variable annotations, and the function body.
- A function without any type parameters is **not** generic; attempting to pass type arguments to a non-generic function is a compile-time error.
- Type parameters may have constraints (`<T extends Constraint>`) and defaults (`<T = Default>`).

**Constraints and limitations.**

- Type parameters that are never used in the signature are unnecessary and discouraged.
- Generic function declarations that always throw infer `void` for the return type unless explicitly annotated `never`; generic function expressions infer `never` in the same situation.
- Generics are erased at compile time; they have no runtime representation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic function declaration (the identity function)

```typescript
// example_generic_declaration.ts
// Step 1: Declare a generic function with type parameter T.
function identity<T>(arg: T): T {
  return arg;
}

// Step 2: Call with an explicit type argument.
let explicit = identity<string>("myString");   // explicit: string

// Step 3: Call with an inferred type argument.
let inferred = identity(42);                  // inferred: number

console.log(explicit.toUpperCase());   // Expected: MYSTRING
console.log(inferred.toFixed(0));      // Expected: 42
```

**Expected Output:**
```
MYSTRING
42
```

**Why this output occurs.** The declaration `identity<T>(arg: T): T` establishes that the return type is the same as the argument type. `identity<string>("myString")` explicitly sets `T = string`, so `.toUpperCase()` is valid. `identity(42)` infers `T = number`, so `.toFixed(0)` is valid. This is the “Hello World” of generics: the type is captured on the input side and restored on the output side.

#### Example 2: Generic arrow function declaration

```typescript
// example_generic_arrow.ts
// Step 1: Declare a generic arrow function.
const firstOrNull = <T>(arr: T[]): T | null => {
  return arr.length === 0 ? null : arr[0];
};

console.log(firstOrNull(["a", "b"]));   // Expected: a
console.log(firstOrNull([]));           // Expected: null
```

**Expected Output:**
```
a
null
```

**Why this output occurs.** The arrow function declares `<T>` before its parameter list, making it generic. `T` is inferred from the array argument. The return type `T | null` accurately reflects that the function may return `null` for an empty array.

### Real-World Cases

- **Utility libraries**: Generic identity and mapping functions are foundational for utility libraries.
- **React hooks**: Generic arrow functions model hooks whose return type depends on a type argument.
- **API wrappers**: `const request = <T>(url: string): Promise<T> => ...` types the response payload generically.

---

## 2. Type Parameter

### Definitions

**Core Definition.** A **type parameter** is a **type-level variable** declared in a generic function’s type parameter list, standing in for a concrete type that will be supplied when the function is used.

**Technical Definition.** Type parameters are declared in angle brackets (`<T>`). They are in scope throughout the function’s signature and body. A type parameter may be **unconstrained**, **constrained** with `extends` (restricting the set of permissible type arguments), or **defaulted** with `=` (providing a fallback when no type argument is supplied). Type parameters enable the function to relate the types of multiple values: if a type parameter appears in two or more positions, it establishes a type relationship between them.

**Beginner-Friendly Explanation.** A type parameter is **a placeholder name for a type**. When you write `<T>`, you are saying “let `T` stand for whatever type is used here.” You can then use `T` in the parameters and return type. Constraints let you say “`T` must have a `length` property,” and defaults let you say “use `string` if no type is given.”

### Purposes

- **To introduce a type variable** that can be used consistently throughout the function signature.
- **To constrain the acceptable type arguments** so that the function can safely access specific members.
- **To provide a sensible fallback type** when callers omit the type argument.
- **To relate the types of multiple parameters and the return value**, preserving type flow.

### Syntax Rules and Structure

```typescript
<T>                          // unconstrained
<T extends Constraint>       // constrained
<T = DefaultType>            // defaulted
<T extends Constraint = Default>  // constrained and defaulted
```

**Component breakdown:**

- `T` — the type parameter name; conventionally single uppercase letters or descriptive names (`TKey`, `TValue`).
- `extends Constraint` — restricts `T` to types assignable to `Constraint`; inside the function, values of type `T` may safely use members of `Constraint`.
- `= DefaultType` — provides a fallback when no type argument is supplied.

**Syntax rules.**

- A constrained type parameter may access members of its constraint inside the function body.
- Required type parameters must not follow defaulted type parameters.
- Constraints may reference other type parameters (e.g., `<T, K extends keyof T>`).
- Type parameters may be documented with the TSDoc `@typeParam` tag (or the JSDoc `@template` tag).

**Constraints and limitations.**

- An unconstrained type parameter permits only operations valid for all types; accessing `.length` or `.toUpperCase()` requires a constraint.
- Defaults should be meaningful; a default that is not useful without specialization creates a false sense of optionality.
- A type parameter that appears only once does not relate anything and is likely unnecessary.

### Annotated Complete Step by Step Code Examples

#### Example 1: Constrained type parameter

```typescript
// example_type_parameter_constraint.ts
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(value: T): T {
  console.log(value.length);   // legal because T has length
  return value;
}

logLength("hello");           // Expected: 5
logLength([1, 2, 3]);         // Expected: 3
// logLength(42);             // Error: number does not satisfy HasLength
```

**Expected Output:**
```
5
3
```

**Why this output occurs.** The constraint `T extends HasLength` guarantees that every `T` has a `length: number` property. Strings and arrays satisfy the constraint; numbers do not, so `logLength(42)` is a compile-time error.

#### Example 2: Default type parameter

```typescript
// example_type_parameter_default.ts
interface Box<T = string> {
  value: T;
}

const defaultBox: Box = { value: "hello" };   // T defaults to string
const numberBox: Box<number> = { value: 42 };

console.log(defaultBox.value);   // Expected: hello
console.log(numberBox.value);    // Expected: 42
```

**Expected Output:**
```
hello
42
```

**Why this output occurs.** `Box` declares `T = string`. When no type argument is supplied, `T` resolves to `string`. When `Box<number>` is used, `T` is `number`, overriding the default.

### Real-World Cases

- **Type-safe property access**: `getProperty<T, K extends keyof T>(obj: T, key: K): T[K]` uses a constrained type parameter to ensure the key is valid.
- **Generic defaults in APIs**: `Options<T = {}>` allows simple call sites to omit type arguments.
- **Documentation**: TSDoc `@typeParam` documents the meaning of each type parameter for consumers.

---

## 3. `<T>` (Angle Bracket Syntax)

### Definitions

**Core Definition.** The **angle bracket syntax** `<T>` is the syntactic construct used to **declare and supply type parameters and type arguments** in TypeScript generics.

**Technical Definition.** Angle brackets serve two related roles: (1) in a **generic declaration**, they introduce type parameters (`function fn<T>(...)`); (2) in a **generic usage**, they supply type arguments (`fn<number>(...)`). The position determines whether the brackets declare a type parameter or provide a type argument. In arrow functions, `<T>` appears before the parameter list; in function declarations and calls, it follows the function name.

**Beginner-Friendly Explanation.** Angle brackets `< >` are how you **write generics**. You put the placeholder type inside them: `<T>`. When you declare a function, you put `<T>` after the name. When you call the function, you can put the actual type inside: `<number>`.

### Purposes

- **To declare type parameters** on functions, interfaces, classes, and type aliases.
- **To supply explicit type arguments** at call sites when inference is insufficient.
- **To distinguish generic declarations from non-generic ones** syntactically.

### Syntax Rules and Structure

```typescript
function fn<T>(arg: T): T { ... }      // declaration: <T> after name
const fn = <T>(arg: T): T => arg;      // arrow declaration: <T> before params
fn<number>(42);                         // usage: <number> after name
```

**Component breakdown:**

- `<T>` — declares a single type parameter named `T`.
- `<T, U>` — declares multiple type parameters separated by commas.
- `<number>` — supplies `number` as the type argument for `T`.

**Syntax rules.**

- In function declarations, `<T>` follows the function name; in arrow functions, it precedes the parameter list.
- In type aliases for generic function types, `<T>` belongs to the **function type**, not the alias: `type Identity = <T>(arg: T) => T`.
- In generic type aliases, `<T>` follows the alias name: `type Identity<T> = (arg: T) => T`. The position matters: `<T>` on the alias name makes the **type** generic; `<T>` on the function makes the **function** generic.

**Constraints and limitations.**

- In `.tsx` files, arrow functions with `<T>` may be parsed as JSX; a trailing comma (`<T,>`) or `extends` constraint resolves the ambiguity.
- Angle brackets do not appear in emitted JavaScript; they are erased.

### Annotated Complete Step by Step Code Examples

#### Example 1: Declaration and usage angle brackets

```typescript
// example_angle_brackets.ts
// Step 1: Declaration with <T>.
function identity<T>(arg: T): T {
  return arg;
}

// Step 2: Usage with explicit <string>.
const a = identity<string>("hello");   // a: string

// Step 3: Usage with inferred type argument.
const b = identity(42);                // b: number

console.log(a);   // Expected: hello
console.log(b);   // Expected: 42
```

**Expected Output:**
```
hello
42
```

**Why this output occurs.** The `<T>` in the declaration introduces the type parameter. The `<string>` in the call supplies an explicit type argument. The second call omits the brackets, and the compiler infers `T = number` from the argument.

#### Example 2: Generic function type vs generic type alias

```typescript
// example_angle_brackets_position.ts
// Type alias for a generic function: T belongs to the function.
type IdentityFn = <T>(arg: T) => T;

// Generic type alias: T belongs to the type.
type Identity<T> = (arg: T) => T;

const fn1: IdentityFn = (arg) => arg;
const fn2: Identity<number> = (arg) => arg;

console.log(fn1("a"));   // Expected: a
console.log(fn2(1));     // Expected: 1
```

**Expected Output:**
```
a
1
```

**Why this output occurs.** `IdentityFn` declares `<T>` on the function type, so each call may use a different type. `Identity<T>` declares `<T>` on the type alias, so `Identity<number>` fixes `T = number` for the entire function type. The position of `<T>` determines which construct is generic.

### Real-World Cases

- **Library signatures**: `<T>` distinguishes generic functions from non-generic ones in declaration files.
- **Type aliases**: `<T>` on function types versus alias names is a common source of confusion in API design.
- **TSX files**: `<T,>` with a trailing comma disambiguates generic arrow functions from JSX.

---

## 4. Multiple Type Parameters

### Definitions

**Core Definition.** **Multiple type parameters** are two or more type variables declared together in a generic function’s type parameter list, separated by commas.

**Technical Definition.** A generic function may declare any number of type parameters: `<T, U>`, `<T, K extends keyof T>`, and so on. Each type parameter is independent and may have its own constraint and default. Multiple parameters allow the function to relate several distinct types — for example, the input type and the output type of a mapping function, or the key and value types of a property accessor.

**Beginner-Friendly Explanation.** Multiple type parameters mean **the function works with more than one placeholder type**. For example, a `pair<T, U>` function takes a `T` and a `U` and returns both together. Each type is tracked separately.

### Purposes

- **To relate multiple distinct types** in a single function signature.
- **To support functions that transform one type into another**, such as `map<T, U>`.
- **To constrain one type parameter by another**, such as `K extends keyof T`.
- **To model pairs, tuples, and key-value structures** with precise types.

### Syntax Rules and Structure

```typescript
function fn<T, U>(a: T, b: U): [T, U] { ... }
function fn<T, K extends keyof T>(obj: T, key: K): T[K] { ... }
```

**Component breakdown:**

- `<T, U>` — two independent type parameters.
- `a: T` — first parameter typed by `T`.
- `b: U` — second parameter typed by `U`.
- `): [T, U]` — return type preserving both types as a tuple.
- `<T, K extends keyof T>` — `K` is constrained to the keys of `T`.

**Syntax rules.**

- Type parameters are separated by commas inside a single pair of angle brackets.
- Each type parameter may have its own constraint and default.
- A later type parameter may reference an earlier one in its constraint (e.g., `<T, K extends keyof T>`).
- Inference determines each type argument independently from the corresponding arguments.

**Constraints and limitations.**

- Too many type parameters can make inference less successful and the signature harder to read.
- A type parameter that does not relate two or more positions is likely unnecessary.
- The order of type parameters matters when explicit type arguments are supplied.

### Annotated Complete Step by Step Code Examples

#### Example 1: Independent multiple type parameters

```typescript
// example_multiple_type_params.ts
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const result = pair("age", 30);
// result: [string, number]

console.log(result[0]);   // Expected: age
console.log(result[1]);   // Expected: 30
```

**Expected Output:**
```
age
30
```

**Why this output occurs.** The compiler infers `T = string` from `"age"` and `U = number` from `30`. The return type is the tuple `[string, number]`, preserving both types precisely.

#### Example 2: Multiple type parameters with constraints

```typescript
// example_multiple_constraints.ts
function merge<T extends object, U extends object>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const merged = merge({ name: "Ada" }, { age: 36 });
console.log(merged);   // Expected: { name: 'Ada', age: 36 }
```

**Expected Output:**
```
{ name: 'Ada', age: 36 }
```

**Why this output occurs.** Both `T` and `U` are constrained to `object`, so only object types are accepted. The return type `T & U` is the intersection, preserving all properties from both arguments.

#### Example 3: Constraint referencing another type parameter

```typescript
// example_keyof_constraint.ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "Ada", age: 36 };
console.log(getProperty(person, "name"));   // Expected: Ada
console.log(getProperty(person, "age"));    // Expected: 36
```

**Expected Output:**
```
Ada
36
```

**Why this output occurs.** `K extends keyof T` ensures that `key` is a valid property name of `obj`. The return type `T[K]` is the indexed access type — the type of the property `K` on `T`. This is a canonical use of multiple type parameters with a constraint.

### Real-World Cases

- **Mapping functions**: `map<T, U>(arr: T[], fn: (item: T) => U): U[]` relates input and output types.
- **Property accessors**: `getProperty<T, K extends keyof T>` ensures type-safe property access.
- **Pair and tuple utilities**: `zip<T, U>(a: T[], b: U[]): [T, U][]` combines two typed arrays.

---

## 5. Generic Function Usage

### Definitions

**Core Definition.** **Generic function usage** refers to **calling or referencing** a generic function, either by supplying **explicit type arguments** in angle brackets or by relying on **type argument inference** from the call’s arguments.

**Technical Definition.** A generic function call may be written as `fn<TypeArgs>(args)` or `fn(args)`. When type arguments are explicit, they are checked against the function’s type parameter constraints and override inference. When they are omitted, TypeScript performs type argument inference using the types of the arguments, the contextual return type, and any other available information. If inference cannot determine a unique type, the inferred type may be a union or `unknown`.

**Beginner-Friendly Explanation.** Using a generic function means **calling it**. You can either tell TypeScript the type explicitly (`identity<number>(42)`) or let it figure out the type from what you pass (`identity(42)`). Most of the time, TypeScript infers the type automatically.

### Purposes

- **To instantiate a generic function** with concrete types.
- **To override inference** when the inferred type is not what you intend.
- **To rely on inference** for concise, readable call sites.
- **To satisfy constraints** by supplying type arguments that meet the function’s requirements.

### Syntax Rules and Structure

```typescript
genericFn<TypeArg>(args);          // explicit type argument
genericFn(args);                    // inferred type argument
const fn: GenericFnType = ...;      // assigning a generic function
```

**Component breakdown:**

- `genericFn<TypeArg>` — supplies an explicit type argument.
- `genericFn(args)` — omits type arguments; the compiler infers them.
- `GenericFnType` — a function type with type parameters, such as `<T>(arg: T) => T`.

**Syntax rules.**

- Explicit type arguments must match the number and constraints of the type parameters.
- When explicit type arguments are supplied, inference is bypassed for those parameters.
- Partial type argument lists are allowed only when remaining parameters have defaults.
- A generic function may be assigned to a variable or parameter of a compatible function type.

**Constraints and limitations.**

- Supplying the wrong number of type arguments is a compile-time error.
- Explicit type arguments that conflict with the argument types produce errors.
- Type arguments are erased at runtime.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit and inferred usage

```typescript
// example_generic_usage.ts
function firstOrNull<ItemType>(arr: ItemType[]): ItemType | null {
  return arr.length === 0 ? null : arr[0];
}

// Explicit type argument.
const explicit = firstOrNull<string>(["Rod", "Jane"]);   // string | null

// Inferred type argument.
const inferred = firstOrNull([1, 2, 3]);                 // number | null

console.log(explicit);   // Expected: Rod
console.log(inferred);   // Expected: 1
```

**Expected Output:**
```
Rod
1
```

**Why this output occurs.** `firstOrNull<string>` explicitly sets `ItemType = string`. `firstOrNull([1, 2, 3])` infers `ItemType = number`. Both calls preserve the element type in the return type.

#### Example 2: Generic function assigned to a typed variable

```typescript
// example_generic_usage_assignment.ts
type IdentityFn = <T>(arg: T) => T;

const identity: IdentityFn = (arg) => arg;

console.log(identity("a"));   // Expected: a
console.log(identity(1));     // Expected: 1
```

**Expected Output:**
```
a
1
```

**Why this output occurs.** `IdentityFn` is a function type with a type parameter on the call signature. Assigning `(arg) => arg` to it is valid because the function is generic. Each call infers its own type argument.

### Real-World Cases

- **React state hooks**: `useState<number>(0)` explicitly types state when inference is insufficient.
- **API clients**: `fetchJson<User>(url)` explicitly types the response payload.
- **Generic utilities**: Most calls to `map`, `filter`, and `reduce` rely on inference, not explicit type arguments.

---

## 6. Type Inference (in Generic Functions)

### Definitions

**Core Definition.** **Type inference in generic functions** is the compiler’s process of **automatically determining type arguments** from the types of the arguments, the contextual return type, and the function’s signature, without requiring explicit type arguments at the call site.

**Technical Definition.** When a generic function is called without type arguments, TypeScript performs **type argument inference**: it unifies the types of the supplied arguments with the parameter types, solving for the type parameters. If a type parameter appears in multiple parameter positions, the inference algorithm may produce a **best common type** or a union. If inference fails or produces an overly wide type, the result may be `unknown` or `any` depending on configuration and constraints. Contextual typing can also propagate an expected type into a generic call.

**Beginner-Friendly Explanation.** Type inference means **TypeScript figures out the type arguments for you**. When you call `identity(42)`, it sees the argument is a number, so it sets `T = number`. You do not have to write `<number>`.

### Purposes

- **To reduce annotation burden** at call sites.
- **To make generic functions feel natural and concise**.
- **To preserve type relationships automatically** through inference from arguments to return types.
- **To support higher-order functions and callbacks** where types flow contextually.

### Syntax Rules and Structure

Inference occurs automatically when type arguments are omitted:

```typescript
function identity<T>(arg: T): T { return arg; }
const x = identity(42);        // T inferred as number
const y = identity("hello");   // T inferred as string
```

**Component breakdown:**

- `identity(42)` — no type arguments; the compiler infers `T` from `42`.
- `identity("hello")` — infers `T` from `"hello"`.

**Syntax rules.**

- Inference uses the **types of the arguments** as the primary source.
- If a type parameter appears in the return type only, it cannot be inferred from arguments; explicit type arguments or contextual typing may be required.
- Constraints participate in inference: if an argument’s type does not satisfy the constraint, inference fails and a compile-time error is reported.
- When multiple arguments contribute to the same type parameter, the inferred type may be a union or a best common type.

**Constraints and limitations.**

- Inference may produce `unknown` or `any` when the argument types provide insufficient information.
- Inference does not perform whole-program analysis; it is local to the call.
- Return-type-only generics cannot be inferred from arguments and require explicit type arguments.

### Annotated Complete Step by Step Code Examples

#### Example 1: Inference from arguments

```typescript
// example_inference_args.ts
function identity<T>(arg: T): T {
  return arg;
}

const num = identity(42);          // T inferred as number
const str = identity("hello");     // T inferred as string

console.log(typeof num);           // Expected: number
console.log(typeof str);           // Expected: string
```

**Expected Output:**
```
number
string
```

**Why this output occurs.** The compiler infers `T` from the type of each argument. `42` is a `number`, so `T = number`; `"hello"` is a `string`, so `T = string`.

#### Example 2: Inference with multiple type parameters

```typescript
// example_inference_multiple.ts
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const result = pair("age", 30);   // [string, number]

console.log(result);   // Expected: [ 'age', 30 ]
```

**Expected Output:**
```
[ 'age', 30 ]
```

**Why this output occurs.** `T` is inferred from `"age"` and `U` from `30`. The return type `[string, number]` reflects both inferred types.

#### Example 3: Inference failure and explicit resolution

```typescript
// example_inference_failure.ts
function parseYAML<T>(input: string): T {
  // implementation omitted
  return JSON.parse(input) as T;
}

// Without explicit type argument, T cannot be inferred from arguments.
// const data = parseYAML("{...}");   // T is unknown

// Explicit type argument resolves the ambiguity.
const user = parseYAML<{ name: string }>('{"name":"Ada"}');
console.log(user.name);   // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `T` appears only in the return type, so inference from the `input: string` argument cannot determine `T`. An explicit type argument is required. This is the return-type-only generic pattern that Google’s style guide advises against creating.

### Real-World Cases

- **Array methods**: `[1, 2, 3].map(x => x * 2)` infers `T = number` and `U = number`.
- **Promise chains**: `fetchJson(url).then(data => ...)` infers the resolved data type from the generic return type.
- **Higher-order functions**: Callback parameter types are inferred contextually from the expected function type.

---

## 7. Generic Function Best Practices

### Definitions

**Core Definition.** **Generic function best practices** are guidelines for writing generic functions that are **safe, readable, and easy to infer**, avoiding unnecessary type parameters, return-type-only generics, and constraints that hinder inference.

**Technical Definition.** Best practices center on the **Golden Rule of Generics**: a type parameter must appear **at least twice** in the function signature to establish a relationship between types. Additional rules include: **push type parameters down** so constraints do not prematurely resolve to `any`; **use fewer type parameters**; **constrain only when necessary**; and **avoid return-type-only generics**, which are effectively unvalidated type assertions.

**Beginner-Friendly Explanation.** Best practices help you write generics that **actually help rather than confuse**. Use generics only when you need to connect the types of inputs and outputs. If a type parameter appears only once, you probably do not need it. Avoid functions where the type is only in the return value — those are just disguised type assertions.

### Purposes

- **To maximize the success of type inference** so callers do not need explicit type arguments.
- **To keep signatures readable** by minimizing the number of type parameters.
- **To preserve genuine type relationships** between inputs and outputs.
- **To avoid creating APIs that give a false sense of type safety**.

### Syntax Rules and Structure

**Key best-practice rules:**

1. **Type parameters should appear at least twice.** If a type parameter appears only once, it relates nothing and is likely unnecessary.
2. **Push type parameters down.** Prefer `firstElement<T>(arr: T[])` over `firstElement<T extends any[]>(arr: T)`; the former preserves the precise element type, while the latter resolves `arr[0]` to `any` through the constraint.
3. **Use fewer type parameters.** A type parameter that does not relate two values is a red flag.
4. **Avoid return-type-only generics.** A generic that appears only in the return type cannot be inferred and is equivalent to an unchecked type assertion.
5. **Constrain only when necessary.** Add an `extends` constraint only when the function body needs to access members of the constraint.
6. **Use descriptive type parameter names** (`TKey`, `TValue`) when the meaning is not obvious.
7. **Document type parameters** with TSDoc `@typeParam` when their meaning is not self-evident.

**Constraints and limitations.**

- Over-constraining can make inference less successful and the API harder to use.
- Defaults that are not meaningful create confusion; avoid them unless the type is genuinely usable without specialization.
- Return-type-only generics should be replaced with `unknown` plus explicit validation or type guards.

### Annotated Complete Step by Step Code Examples

#### Example 1: Golden Rule — type parameter must appear twice

```typescript
// example_best_golden_rule.ts
// GOOD: T appears in parameter and return type.
function identity<T>(arg: T): T {
  return arg;
}

// BAD: T appears only once (return type).
// function parseYAML<T>(input: string): T { ... }
// Equivalent to returning any; T cannot be inferred from arguments.

console.log(identity("hello"));   // Expected: hello
```

**Expected Output:**
```
hello
```

**Why this output occurs.** `identity` relates the argument type to the return type through `T`. `parseYAML<T>` would appear to return a typed value, but `T` is not connected to any argument, so the caller can assert any type without validation — a dangerous anti-pattern.

#### Example 2: Push type parameters down

```typescript
// example_best_push_down.ts
// GOOD: type parameter is the element type.
function firstElement1<T>(arr: T[]): T {
  return arr[0];
}

// BAD: constraint resolves arr[0] through the constraint type.
function firstElement2<T extends any[]>(arr: T): any {
  return arr[0];
}

const a = firstElement1([1, 2, 3]);   // a: number (good)
const b = firstElement2([1, 2, 3]);   // b: any (bad)

console.log(typeof a);   // Expected: number
console.log(typeof b);   // Expected: number (runtime), but statically any
```

**Expected Output:**
```
number
number
```

**Why this output occurs.** `firstElement1` uses `T` as the element type, so inference produces `number`. `firstElement2` constrains `T extends any[]` but returns `arr[0]`, which the compiler resolves through the constraint as `any`, losing the element type.

#### Example 3: Use fewer type parameters

```typescript
// example_best_fewer.ts
// BAD: F does not relate two values.
function filter2<T, F extends (arg: T) => boolean>(arr: T[], func: F): T[] {
  return arr.filter(func);
}

// GOOD: only T is needed.
function filter1<T>(arr: T[], func: (arg: T) => boolean): T[] {
  return arr.filter(func);
}

console.log(filter1([1, 2, 3], (n) => n > 1));   // Expected: [2, 3]
```

**Expected Output:**
```
[2, 3]
```

**Why this output occurs.** `F` in `filter2` does not relate any two values; it only constrains the callback type. It adds an unnecessary type parameter that callers would have to specify manually. `filter1` expresses the same constraint without the extra parameter.

### Real-World Cases

- **Library APIs**: Utility libraries use the Golden Rule to keep generics inferable and ergonomic.
- **API clients**: Return-type-only generics are avoided in favor of `unknown` plus validation.
- **Code review**: ESLint rules such as `@typescript-eslint/no-unnecessary-type-parameters` enforce the Golden Rule.

---

## References

- TypeScript Handbook — Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- Microsoft TypeScript Handbook — Guidelines for Writing Good Generic Functions (Push Type Parameters Down, Use Fewer Type Parameters) - https://microsoft.github.io/TypeScript-New-Handbook/everything/#guidelines-for-writing-good-generic-functions
- Google TypeScript Style Guide — Return type only generics - https://google.github.io/styleguide/tsguide.html
- Effective TypeScript — Avoid Unnecessary Type Parameters (Golden Rule) - https://github.com/marius-townhouse/effective-typescript-skills
- TypeScript Advanced Generics — Constraints with `extends` and multiple type parameters - https://github.com/Compile-N-Run/Compile-N-Run
- Total TypeScript — Generic Functions (Type parameters and arrow function syntax) - https://www.totaltypescript.com/books/total-typescript-essentials/utility-folder-development-in-typescript
- TSDoc — `@typeParam` Tag - https://tsdoc.org/pages/tags/typeparam/
- TypeScript Handbook — Type Argument Inference - https://www.typescriptlang.org/docs/handbook/type-inference.html