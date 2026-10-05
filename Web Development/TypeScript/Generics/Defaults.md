# TypeScript Generic Defaults: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Generic defaults (also called generic parameter defaults or default type parameters) are a TypeScript feature that allows you to specify a fallback type for a generic type parameter. When a type argument is not provided (and cannot be inferred), TypeScript uses the default type instead. This makes the type parameter optional at the use site.

**Technical Definition**
A generic parameter default is declared using an equals sign after the type parameter's name (and after any constraint): `<T = DefaultType>` or `<T extends Constraint = DefaultType>`. A type parameter is deemed optional if it has a default. When resolving type arguments, TypeScript applies defaults from left to right: if a default refers to an earlier type parameter, that parameter's type argument is used; if it refers to a later parameter, the empty object type `{}` is used. The default type must satisfy the type parameter's constraint, if one exists. Unspecified type parameters resolve to their defaults, and if inference cannot choose a candidate, the default type is inferred.

**Beginner-Friendly Explanation**
Generic defaults let you give a generic type a "backup" type. If you write `interface Box<T = string>`, then `Box` without a type argument is the same as `Box<string>`. You only need to specify a type when you want something other than the default. This is like function parameter defaults: if you don't provide an argument, the default is used. Generic defaults make APIs easier to use—simple cases don't need to specify types, but complex cases can still override the default.

### Key Characteristics

- **Optional type arguments**: A type parameter with a default is optional at the use site.
- **Placement rule**: Required type parameters must not follow optional (defaulted) type parameters.
- **Constraint satisfaction**: The default type must satisfy the type parameter's constraint.
- **Left-to-right resolution**: Defaults are applied in order; defaults can reference earlier parameters.
- **Reduced verbosity**: Simple use cases don't need to specify type arguments.
- **Mergeable**: A class or interface declaration that merges with an existing declaration may introduce a default.

### Prerequisites

- Basic knowledge of TypeScript generics
- Familiarity with generic constraints (`extends`)
- Understanding of type parameters and type arguments
- Familiarity with interfaces, classes, and type aliases

### Related Programming Areas

- **API Design**: Making generic APIs easier to consume
- **Type-Level Programming**: Conditional defaults, constraint interactions
- **Library Development**: Reducing boilerplate for consumers
- **Utility Types**: Built-in utility types use defaults (e.g., `Promise<T = unknown>`)
- **Function Defaults**: Analogous to default function parameters

### Core Concepts / Features

1. Default Type Parameter Syntax and Placement Rules
2. Optional Generic Arguments and Their Interaction with Required Parameters
3. Generic API Design, Sensible Defaults, and Minimizing Developer Verbosity
4. Interaction Between Type Parameter Constraints and Defaults (`<T extends object = {}>`)


## 1. Default Type Parameter Syntax and Placement Rules

### Definitions

**Core Definition**
Default type parameter syntax uses an equals sign after the type parameter name (and after any constraint) to specify a fallback type. The placement rule requires that required type parameters must not follow optional (defaulted) type parameters.

**Technical Definition**
The syntax for a generic parameter default is `<TypeParameter extends Constraint = DefaultType>`, where both the constraint and default are optional. The `=` introduces the default type. TypeScript applies defaults from left to right during type argument resolution. A type parameter is optional if it has a default. Required type parameters (those without defaults) must not follow optional ones. When a default references an earlier type parameter, that parameter's type argument is used; when it references a later parameter, the empty object type `{}` is used to avoid circularity checks.

**Beginner-Friendly Explanation**
You write generic defaults like this: `<T = string>`. The `= string` is the default. If you have a constraint, the default goes after it: `<T extends object = {}>`. The rule about placement is simple: once you give a type parameter a default, all following type parameters must also have defaults. You can't have a required parameter after an optional one—just like function parameters. This keeps the type argument list unambiguous.

### Purposes

- To make type parameters optional at the use site.
- To provide sensible fallback types for common use cases.
- To reduce the need for explicit type arguments in simple scenarios.
- To enable partial type argument specification (some explicit, some defaulted).
- To align generic defaults with function parameter defaults.

### Syntax Rules and Structure

**General Syntax: Default Without Constraint**

```typescript
interface InterfaceName<T = DefaultType> {
  property: T;
}
```

**Component Breakdown**
- `<T = DefaultType>`: The type parameter `T` defaults to `DefaultType`.
- If no type argument is provided, `T` is `DefaultType`.

**General Syntax: Default With Constraint**

```typescript
interface InterfaceName<T extends Constraint = DefaultType> {
  property: T;
}
```

**Component Breakdown**
- `<T extends Constraint = DefaultType>`: The default must satisfy `Constraint`.

**General Syntax: Multiple Defaults**

```typescript
interface Pair<T = string, U = number> {
  first: T;
  second: U;
}
```

**Component Breakdown**
- Both parameters have defaults; both are optional.

**General Syntax: Mixed Required and Optional**

```typescript
interface Result<T, E = Error> {
  data: T;
  error: E;
}
```

**Component Breakdown**
- `T` is required; `E` is optional and defaults to `Error`.
- Required parameters must come before optional ones.

**Syntax Rules**

- The default is introduced by `=` after the type parameter name.
- If a constraint exists, the default follows the constraint: `<T extends C = D>`.
- A type parameter with a default is optional.
- Required type parameters must not follow optional type parameters.
- Defaults are applied left to right.
- A default referencing an earlier parameter uses that parameter's type argument.
- A default referencing a later parameter uses `{}`.
- The default must satisfy the constraint, if one exists.

**Constraints and Limitations**

- Required type parameters cannot follow optional ones (compile error).
- A default that does not satisfy its constraint is a compile error.
- Defaults cannot reference later type parameters directly (they resolve to `{}`).
- Interface/class declaration merging has specific rules for defaults.
- Defaults are erased at runtime.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Default Type Parameter

```typescript
// Step 1: Define an interface with a default type parameter.
interface Box<T = string> {
  value: T;
}

// Step 2: Use without a type argument — uses the default.
const stringBox: Box = { value: "hello" };
// Box<string> — T defaults to string.

// Step 3: Use with an explicit type argument — overrides the default.
const numberBox: Box<number> = { value: 42 };
// Box<number> — T is number.

// Step 4: Type safety is preserved in both cases.
const str: string = stringBox.value;    // ✅
const num: number = numberBox.value;    // ✅
// const wrong: number = stringBox.value; // ❌ Error.

console.log(stringBox.value);  // "hello"
console.log(numberBox.value);  // 42

// Step 5: Defaults work with multiple parameters.
interface Pair<T = string, U = number> {
  first: T;
  second: U;
}

const defaultPair: Pair = { first: "a", second: 1 };        // Pair<string, number>
const mixedPair: Pair<boolean> = { first: true, second: 2 }; // Pair<boolean, number>
const fullPair: Pair<boolean, string> = { first: true, second: "b" }; // Pair<boolean, string>

console.log(defaultPair);  // { first: 'a', second: 1 }
console.log(mixedPair);    // { first: true, second: 2 }
console.log(fullPair);     // { first: true, second: 'b' }
```

**Expected Output:**
```
hello
42
{ first: 'a', second: 1 }
{ first: true, second: 2 }
{ first: true, second: 'b' }
```

**Why This Output Occurs:** The `Box<T = string>` interface defaults `T` to `string`. When no type argument is provided, `T` is `string`. When `<number>` is provided, `T` is `number`. The `Pair` interface demonstrates multiple defaults and partial specification.

#### Example 2: Placement Rule — Required After Optional Is an Error

```typescript
// Step 1: Valid placement — required before optional.
interface ValidResult<T, E = Error> {
  data: T;
  error: E;
}

const result: ValidResult<string> = {
  data: "hello",
  error: new Error("Something failed"),
};

console.log(result.data);  // "hello"

// Step 2: Invalid placement — required after optional is a compile error.
// interface InvalidResult<T = string, U> { }
// ❌ Error: Required type parameters must not follow optional type parameters.

// Step 3: The error occurs at the declaration site.
// TypeScript will not compile the interface.

// Step 4: Workaround — give the second parameter a default too.
interface FixedResult<T = string, U = number> {
  first: T;
  second: U;
}

const fixed: FixedResult = { first: "a", second: 1 };
console.log(fixed);  // { first: 'a', second: 1 }
```

**Expected Output:**
```
hello
{ first: 'a', second: 1 }
```

**Why This Output Occurs:** The `ValidResult<T, E = Error>` interface is valid because the required parameter `T` comes before the optional parameter `E`. The `InvalidResult<T = string, U>` interface is a compile error because `U` is required but follows an optional parameter. The `FixedResult` interface gives both parameters defaults, resolving the issue.

### Real-World Cases

**Case 1: API Response Types**
`ApiResponse<T, E = Error>` allows callers to omit the error type when the default is acceptable.

**Case 2: Event Handlers**
`EventHandler<T = Event>` defaults to the base event type, with specific events overriding.

**Case 3: Configuration Objects**
`Config<T = unknown>` defaults to `unknown` for flexible configuration values.

**Case 4: Promise-Like Types**
`Promise<T = unknown>` uses a default to allow `Promise` without a type argument (though this is discouraged).


## 2. Optional Generic Arguments and Their Interaction with Required Parameters

### Definitions

**Core Definition**
Optional generic arguments are type arguments that can be omitted at the use site because the corresponding type parameter has a default. Required generic arguments are those without defaults and must be provided. The interaction rule ensures that optional arguments can only appear after required arguments in the type parameter list.

**Technical Definition**
When specifying type arguments, you are only required to specify arguments for the required type parameters. Unspecified type parameters resolve to their default types. If a default type is specified and inference cannot choose a candidate, the default type is inferred. This enables partial specification: some type arguments explicit, others defaulted. The rule "required type parameters must not follow optional type parameters" ensures that the mapping between provided type arguments and type parameters is unambiguous from left to right.

**Beginner-Friendly Explanation**
Optional generic arguments work like optional function parameters: you can omit them if they have defaults. For example, `Result<T, E = Error>` lets you write `Result<string>` instead of `Result<string, Error>`. The rule is that once a parameter is optional (has a default), all following parameters must also be optional. This prevents ambiguity: if you provide two type arguments, TypeScript knows they go to the first two parameters. If you provide one, it goes to the first, and the rest use defaults.

### Purposes

- To allow partial specification of type arguments.
- To reduce verbosity when defaults are acceptable.
- To enable type inference to fall back to defaults.
- To provide a smooth API for both simple and complex use cases.
- To align generic argument specification with function argument specification.

### Syntax Rules and Structure

**General Syntax: Partial Type Argument Specification**

```typescript
interface Result<T, E = Error> {
  data: T;
  error: E;
}

const success: Result<string> = { data: "ok", error: null as any };
// E defaults to Error.
```

**Component Breakdown**
- `Result<string>`: Only `T` is specified; `E` uses its default.

**General Syntax: Omitted Type Arguments Fall Back to Defaults**

```typescript
function createBox<T = string>(value: T): Box<T> {
  return { value };
}

const box = createBox("hello");  // T inferred as string (not default)
```

**Component Breakdown**
- Inference takes priority over the default.

**General Syntax: Inference Failure Falls Back to Default**

```typescript
function createArray<T = number>(): T[] {
  return [];
}

const empty = createArray();  // T is number (default, since nothing to infer)
```

**Component Breakdown**
- When inference cannot choose a candidate, the default is used.

**Syntax Rules**

- Only required type parameters must be specified.
- Unspecified type parameters resolve to their defaults.
- Inference takes priority over defaults when a candidate can be inferred.
- If inference fails, the default is used.
- Required parameters must precede optional ones.
- Type arguments are matched to type parameters left to right.

**Constraints and Limitations**

- If a required parameter follows an optional one, the declaration is a compile error.
- Defaults do not act as constraints; they are fallbacks.
- Inference can sometimes produce a wider type than the default (e.g., `string` instead of a literal).
- Defaults are not enforced at runtime.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Partial Specification

```typescript
// Step 1: Define an interface with one required and one optional parameter.
interface ApiResponse<T, E = Error> {
  data: T;
  error: E;
}

// Step 2: Specify only the required parameter.
const success: ApiResponse<string> = {
  data: "hello",
  error: null as any,  // E defaults to Error
};

// Step 3: Specify both parameters.
const customError: ApiResponse<string, string> = {
  data: "hello",
  error: "Something failed",  // E is string
};

// Step 4: Type safety is preserved.
const data: string = success.data;        // ✅
const error: Error = success.error;       // ✅ (E is Error)
const customErrorMsg: string = customError.error; // ✅ (E is string)

console.log(success.data);         // "hello"
console.log(customError.error);    // "Something failed"
```

**Expected Output:**
```
hello
Something failed
```

**Why This Output Occurs:** The `ApiResponse<T, E = Error>` interface has `T` as required and `E` as optional (defaulting to `Error`). Specifying only `string` makes `E` default to `Error`. Specifying both `string, string` overrides the default.

#### Example 2: Inference vs. Default

```typescript
// Step 1: Define a generic function with a default.
function wrapInArray<T = string>(value?: T): T[] {
  return value === undefined ? [] : [value];
}

// Step 2: Inference takes priority when a value is provided.
const strings = wrapInArray("hello");  // T inferred as string
const numbers = wrapInArray(42);       // T inferred as number

console.log(strings);  // ["hello"]
console.log(numbers);  // [42]

// Step 3: Default is used when inference fails (no argument).
const empty = wrapInArray();  // T defaults to string
console.log(empty);  // []

// Step 4: Explicit type argument overrides both.
const booleans = wrapInArray<boolean>(true);
console.log(booleans);  // [true]

// Step 5: The default does not act as a constraint.
// wrapInArray(42) works even though the default is string.
```

**Expected Output:**
```
["hello"]
[42]
[]
[true]
```

**Why This Output Occurs:** When an argument is provided, TypeScript infers `T` from it (e.g., `string` or `number`). When no argument is provided, inference fails, and the default (`string`) is used. An explicit type argument overrides both.

### Real-World Cases

**Case 1: Result Types**
`Result<T, E = Error>` allows callers to omit the error type when the default is appropriate.

**Case 2: React Hooks**
`useState<T = undefined>()` defaults to `undefined` when no initial value is provided.

**Case 3: Database Clients**
`Query<T = unknown>` defaults to `unknown` for flexible query results.

**Case 4: Event Emitters**
`EventEmitter<Events = Record<string, unknown>>` defaults to a generic event map.


## 3. Generic API Design, Sensible Defaults, and Minimizing Developer Verbosity

### Definitions

**Core Definition**
Generic API design with defaults involves choosing default types that make the API useful without explicit type arguments while still allowing full customization. Sensible defaults reduce verbosity for common cases and make the API approachable for new users.

**Technical Definition**
In generic API design, defaults should be chosen so that the type is meaningfully usable without type arguments. A default that makes the type useless (e.g., `unknown` when the type requires specific operations) creates a false sense of optionality. Good defaults represent a genuine "no-specialization" mode—a common case that works without further configuration. Libraries use defaults like `T = unknown` or `T = void` for options objects, allowing simple uses to be simple while keeping complex uses possible. The default should satisfy the constraint and be the most common or least surprising type for the context.

**Beginner-Friendly Explanation**
When you design a generic API, you should pick defaults that make the API easy to use without type arguments. For example, if you have an options object that's usually just `{}`, defaulting to `{}` makes sense. If you have a result type where errors are usually `Error`, defaulting to `Error` saves typing. But don't add a default just to make the type parameter "optional"—if the type is useless without a specific type, a default is misleading. The goal is to make simple cases simple and complex cases possible.

### Purposes

- To make generic APIs approachable for common use cases.
- To reduce verbosity when defaults are acceptable.
- To provide a "no-specialization" mode for the API.
- To avoid forcing consumers to specify types they don't care about.
- To keep the API flexible for advanced use cases.

### Syntax Rules and Structure

**General Syntax: API with Sensible Default**

```typescript
interface ApiClientOptions<T = unknown> {
  baseUrl: string;
  transform?: (data: unknown) => T;
}
```

**Component Breakdown**
- `T = unknown`: The default allows simple use without a type argument.

**General Syntax: Result Type with Error Default**

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };
```

**Component Breakdown**
- `E = Error`: The common error type is the default.

**General Syntax: Options Object with Empty Default**

```typescript
type RequestOptions<T extends object = {}> = {
  headers?: Record<string, string>;
} & T;
```

**Component Breakdown**
- `T extends object = {}`: Empty options by default.

**Syntax Rules**

- Defaults should be the most common or least surprising type.
- The default must satisfy the constraint.
- Avoid defaults that make the type useless without a type argument.
- Use `unknown` as a safe default when the type is truly unconstrained.
- Use `void` for optional generic parameters that are not used.
- Document the default in API documentation.

**Constraints and Limitations**

- A default that is too wide (e.g., `unknown`) may hide type errors.
- A default that is too narrow (e.g., `string`) may be too restrictive.
- Defaults do not act as constraints; they are fallbacks.
- Overusing defaults can make APIs confusing when the default is not obvious.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Result Type with Sensible Default

```typescript
// Step 1: Define a Result type with a default error type.
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

// Step 2: Use with the default error type.
function parseNumber(input: string): Result<number> {
  const parsed = Number(input);
  if (isNaN(parsed)) {
    return { success: false, error: new Error("Invalid number") };
  }
  return { success: true, data: parsed };
}

// Step 3: Use with a custom error type.
function parseNumberCustom(input: string): Result<number, string> {
  const parsed = Number(input);
  if (isNaN(parsed)) {
    return { success: false, error: "Invalid number" };
  }
  return { success: true, data: parsed };
}

// Step 4: Test both.
const result1 = parseNumber("42");
console.log(result1);  // { success: true, data: 42 }

const result2 = parseNumber("abc");
console.log(result2);  // { success: false, error: Error: Invalid number }

const result3 = parseNumberCustom("abc");
console.log(result3);  // { success: false, error: 'Invalid number' }
```

**Expected Output:**
```
{ success: true, data: 42 }
{ success: false, error: Error: Invalid number }
{ success: false, error: 'Invalid number' }
```

**Why This Output Occurs:** The `Result<T, E = Error>` type defaults `E` to `Error`. Callers who don't specify an error type get `Error` by default. Callers who want a custom error type can specify it explicitly.

#### Example 2: Options Object with Empty Default

```typescript
// Step 1: Define an options type with an empty object default.
type RequestOptions<T extends object = {}> = {
  timeout?: number;
  retries?: number;
} & T;

// Step 2: Use with no extra options — T defaults to {}.
const basicRequest: RequestOptions = {
  timeout: 5000,
};
console.log(basicRequest);  // { timeout: 5000 }

// Step 3: Use with custom options.
interface AuthOptions {
  authToken: string;
  userId: number;
}

const authRequest: RequestOptions<AuthOptions> = {
  timeout: 5000,
  authToken: "token-123",
  userId: 1,
};
console.log(authRequest);
// { timeout: 5000, authToken: 'token-123', userId: 1 }

// Step 4: The default does not constrain — any object works.
const customRequest: RequestOptions<{ custom: boolean }> = {
  custom: true,
};
console.log(customRequest);  // { custom: true }
```

**Expected Output:**
```
{ timeout: 5000 }
{ timeout: 5000, authToken: 'token-123', userId: 1 }
{ custom: true }
```

**Why This Output Occurs:** The `RequestOptions<T extends object = {}>` type defaults `T` to `{}`. When no type argument is provided, the options object has only the base properties. When a type argument is provided, the extra properties are added via intersection.

### Real-World Cases

**Case 1: HTTP Clients**
`RequestOptions<T = {}>` allows simple requests without extra options and complex requests with custom headers or body types.

**Case 2: State Management**
`Store<T = unknown>` defaults to `unknown` for simple stores, with specific types for domain stores.

**Case 3: Form Libraries**
`FormValues<T = Record<string, unknown>>` defaults to a generic record for simple forms.

**Case 4: Query Builders**
`Query<T = unknown>` defaults to `unknown` for flexible query results, with specific types for typed queries.


## 4. Interaction Between Type Parameter Constraints and Defaults (`<T extends object = {}>`)

### Definitions

**Core Definition**
When a type parameter has both a constraint and a default, the default must satisfy the constraint. The syntax is `<T extends Constraint = DefaultType>`. This allows you to specify both a minimum requirement (the constraint) and a fallback type (the default) that meets that requirement.

**Technical Definition**
The constraint and default interact through the assignability rule: the default type must be assignable to the constraint. If the default does not satisfy the constraint, TypeScript produces a compile error at the declaration site. The constraint limits what types can be used as type arguments; the default provides a fallback when no type argument is provided. When a type argument is provided, it must satisfy the constraint (not necessarily the default). When no type argument is provided, the default is used, and it must satisfy the constraint. The pattern `<T extends object = {}>` is a common idiom: the constraint requires an object type, and the default `{}` satisfies it.

**Beginner-Friendly Explanation**
When you have both a constraint and a default, the default must follow the rules of the constraint. If your constraint says "T must be an object," your default can't be a string—it must also be an object. The pattern `<T extends object = {}>` is common: it says "T must be an object, and if you don't specify one, use an empty object." The default `{}` is an object, so it satisfies the constraint. If you tried `<T extends object = string>`, TypeScript would give you an error because `string` is not an object.

### Purposes

- To provide a default type that meets the constraint's requirements.
- To allow simple use cases without type arguments while enforcing a minimum shape.
- To use `{}` as a safe default for object-constrained type parameters.
- To document the most common type for a constrained parameter.
- To enable partial specification with constrained defaults.

### Syntax Rules and Structure

**General Syntax: Constraint with Default**

```typescript
interface InterfaceName<T extends Constraint = DefaultType> {
  property: T;
}
```

**Component Breakdown**
- `T extends Constraint`: The constraint.
- `= DefaultType`: The default, which must satisfy `Constraint`.

**General Syntax: Object Constraint with Empty Default**

```typescript
function functionName<T extends object = {}>(param: T): T {
  return param;
}
```

**Component Breakdown**
- `T extends object = {}`: The default `{}` satisfies `object`.

**General Syntax: Multiple Parameters with Constraints and Defaults**

```typescript
interface Pair<T extends object = {}, U extends object = {}> {
  first: T;
  second: U;
}
```

**Component Breakdown**
- Both parameters have object constraints and empty object defaults.

**Syntax Rules**

- The default must satisfy the constraint (assignability check).
- If the default does not satisfy the constraint, it's a compile error.
- The constraint limits what type arguments can be provided.
- The default provides a fallback when no type argument is provided.
- When a type argument is provided, it must satisfy the constraint, not necessarily the default.
- The default can reference earlier type parameters.

**Constraints and Limitations**

- A default that does not satisfy its constraint is a compile error.
- The default does not act as a constraint (it's a fallback, not a restriction).
- If the constraint is very specific, the default must be equally specific.
- The `{}` default satisfies most object constraints because it's the empty object type.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Object Constraint with Empty Object Default

```typescript
// Step 1: Define a type with an object constraint and empty object default.
type ComponentProps<T extends object = {}> = {
  className?: string;
  children?: unknown;
} & T;

// Step 2: Use without a type argument — T defaults to {}.
const basicProps: ComponentProps = {
  className: "container",
};
console.log(basicProps);  // { className: 'container' }

// Step 3: Use with a type argument that satisfies the constraint.
interface ButtonProps {
  onClick: () => void;
  variant: "primary" | "secondary";
}

const buttonProps: ComponentProps<ButtonProps> = {
  className: "btn",
  onClick: () => console.log("Clicked"),
  variant: "primary",
};
console.log(buttonProps.variant);  // "primary"

// Step 4: A default that does not satisfy the constraint is a compile error.
// type Invalid<T extends object = string> = T;
// ❌ Error: Type 'string' does not satisfy the constraint 'object'.

// Step 5: The default does not constrain the type argument.
// ComponentProps<string> is a compile error because string does not satisfy object.
// ComponentProps<{ custom: boolean }> is fine.
const customProps: ComponentProps<{ custom: boolean }> = {
  custom: true,
};
console.log(customProps);  // { custom: true }
```

**Expected Output:**
```
{ className: 'container' }
primary
{ custom: true }
```

**Why This Output Occurs:** The `ComponentProps<T extends object = {}>` type constrains `T` to objects and defaults to `{}`. When no type argument is provided, `T` is `{}`, so only the base properties are present. When a type argument is provided, it must be an object (satisfying the constraint). A string type argument is a compile error because it violates the constraint.

#### Example 2: Constraint and Default with Dependencies

```typescript
// Step 1: Define a type where the default references an earlier parameter.
interface Wrapper<T, U = T> {
  value: T;
  wrapped: U;
}

// Step 2: Use with one type argument — U defaults to T.
const wrapped: Wrapper<string> = {
  value: "hello",
  wrapped: "world",  // U is string (defaults to T)
};
console.log(wrapped);  // { value: 'hello', wrapped: 'world' }

// Step 3: Use with two type arguments — U is overridden.
const custom: Wrapper<string, number> = {
  value: "hello",
  wrapped: 42,
};
console.log(custom);  // { value: 'hello', wrapped: 42 }

// Step 4: Constraint with default referencing earlier parameter.
interface Result<T, E extends Error = Error> {
  data: T;
  error: E;
}

const defaultResult: Result<string> = {
  data: "ok",
  error: new Error("Failed"),  // E defaults to Error
};
console.log(defaultResult.data);  // "ok"

// Step 5: Custom error type satisfies the constraint.
class CustomError extends Error {
  code: number = 500;
}

const customResult: Result<string, CustomError> = {
  data: "ok",
  error: new CustomError("Failed"),  // E is CustomError
};
console.log(customResult.error.code);  // 500
```

**Expected Output:**
```
{ value: 'hello', wrapped: 'world' }
{ value: 'hello', wrapped: 42 }
ok
500
```

**Why This Output Occurs:** The `Wrapper<T, U = T>` type defaults `U` to `T` (the earlier parameter). The `Result<T, E extends Error = Error>` type constrains `E` to `Error` and defaults it to `Error`. A custom error type that extends `Error` satisfies the constraint.

### Real-World Cases

**Case 1: React Component Props**
`ComponentProps<T extends object = {}>` allows components to accept extra props via the generic parameter while defaulting to no extra props.

**Case 2: API Request Options**
`RequestOptions<T extends object = {}>` allows custom request options while defaulting to an empty object.

**Case 3: State Containers**
`Store<T extends object = {}>` constrains state to objects and defaults to an empty object.

**Case 4: Form Values**
`FormValues<T extends object = {}>` constrains form values to objects and defaults to an empty object.

---

## References

- TypeScript Handbook: Generic Parameter Defaults — https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-parameter-defaults
- TypeScript 2.3 Release Notes: Generic Parameter Defaults — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-3.html#generic-parameter-defaults
- TypeScript PR #13487: Adds Support for Type Parameter Defaults — https://github.com/microsoft/TypeScript/pull/13487
- Total TypeScript: Specify a Default Type Parameter — https://www.totaltypescript.com/workshops/typescript-pro-essentials/designing-your-types/default-type-parameters-in-generics/solution
- TypeScript Playground: Generic Parameter Defaults — https://www.typescriptlang.org/play/typescript/generics.ts.html
- Stack Overflow: Generic Defaults Trigger Error for Dependent Generic Constraint Variable — https://stackoverflow.com/questions/78456231
- Stack Overflow: TypeScript Unable to Infer Generic Parameter Default with Optional Argument — https://stackoverflow.com/questions/78880939
- TypeScript Issue #30480: Sub-class Generics with Super-class Constraint/Default Generics — https://github.com/microsoft/TypeScript/issues/30480
- TypeScript Handbook: Generic Constraints — https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints
- TypeScript Handbook: Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Deep Dive: Generics — https://basarat.gitbook.io/typescript/type-system/generics
- Effective TypeScript: Item 14 — Use Type Operations and Generics to Avoid Repeating Yourself