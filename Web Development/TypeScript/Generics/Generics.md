## Generics Fundamentals

**Topic Overview.** **Generics** are TypeScript's primary mechanism for creating **reusable, type-safe components** that operate over a variety of types rather than a single one. A generic component declares one or more **type parameters** — placeholders for types supplied later — and uses them throughout its signature and body. When the component is used, **type arguments** either are provided explicitly or are inferred by the compiler from the values passed in. Unlike using `any`, generics **preserve type information**: the type relationship between inputs and outputs remains intact, so callers receive precise types rather than `any`. Generics are the foundation for TypeScript's most powerful type-level abstractions, including utility types (`Array<T>`, `Promise<T>`, `Record<K, V>`), constraint-driven APIs, and variance-aware designs.

---

## 1. Generics

### Definitions

**Core Definition.** **Generics** are a language feature that allows functions, interfaces, classes, and type aliases to be **parameterized over types**, so that a single declaration can work over many types while preserving type safety.

**Technical Definition.** Generics introduce **type parameters** — type-level variables declared in angle brackets (`<T>`). A generic declaration is a **type constructor**: it takes type arguments and produces a concrete type. In TypeScript, generics are checked at compile time and **erased** at runtime, like all type-level constructs. The type system uses **type argument inference** to determine type arguments from usage when they are not explicitly provided.

**Beginner-Friendly Explanation.** Generics let you write **one piece of code that works with many types** without losing track of which type is which. Instead of writing separate functions for numbers, strings, and objects, you write a function with a placeholder type (`<T>`) that TypeScript fills in when you call it. The placeholder ensures that whatever type goes in is the same type that comes out.

### Purposes

- **To enable code reuse across types** without sacrificing type safety or resorting to `any`.
- **To preserve type relationships** between inputs and outputs, so the compiler knows that an `identity<number>(42)` returns a `number`, not `any`.
- **To express type-level abstractions** that are checked at compile time and erased at runtime.
- **To provide the foundation for generic libraries and utility types** such as `Array<T>`, `Promise<T>`, and `Record<K, V>`.

### Syntax Rules and Structure

**General syntax (generic function):**

```typescript
function functionName<TypeParam>(param: TypeParam): TypeParam {
  return param;
}
```

**Component breakdown:**

- `functionName` — the function identifier.
- `<TypeParam>` — the **type parameter list**; declares one or more type variables.
- `param: TypeParam` — a parameter whose type is the type parameter.
- `): TypeParam` — the return type, preserving the type relationship.

**Syntax rules.**

- Type parameters are declared in angle brackets immediately after the function, class, or interface name.
- Type arguments may be supplied explicitly (`identity<number>(42)`) or inferred from arguments (`identity(42)`).
- Generics are **erased**; no runtime type information survives compilation.
- Generic enums and namespaces are **not supported**; generics apply to functions, interfaces, classes, and type aliases.

**Constraints and limitations.**

- A generic type parameter that is never used in the declaration is a code smell and is discouraged.
- Generics cannot be used with `enum` or `namespace` declarations.
- Without constraints, a type parameter is treated as if it could be any type, which restricts operations on values of that type.

### Annotated Complete Step by Step Code Examples

#### Example 1: The identity function — generics vs `any`

```typescript
// example_generics_identity.ts
// Step 1: Generic identity function.
function identity<T>(arg: T): T {
  return arg;
}

// Step 2: Explicit type argument.
let output1 = identity<string>("myString");  // output1: string

// Step 3: Inferred type argument.
let output2 = identity(42);                  // output2: number

// Step 4: Contrast with any (loses type information).
function identityAny(arg: any): any {
  return arg;
}
let output3 = identityAny("myString");       // output3: any

console.log(output1.toUpperCase());   // Expected: MYSTRING
console.log(output2.toFixed(0));      // Expected: 42
// console.log(output3.toUpperCase()); // would compile with any but loses safety
```

**Expected Output:**
```
MYSTRING
42
```

**Why this output occurs.** `identity<T>` preserves the argument’s type in its return type. `identity<string>("myString")` returns `string`, so `.toUpperCase()` is valid. `identity(42)` infers `T = number`, so `.toFixed(0)` is valid. `identityAny` returns `any`, erasing the type relationship and disabling checking on the result.

#### Example 2: Generic array wrapper

```typescript
// example_generics_array.ts
// Step 1: Generic function operating on arrays.
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

// Step 2: Call with different array types.
const firstNum = firstElement([1, 2, 3]);        // number | undefined
const firstStr = firstElement(["a", "b"]);       // string | undefined

console.log(firstNum);   // Expected: 1
console.log(firstStr);   // Expected: a
```

**Expected Output:**
```
1
a
```

**Why this output occurs.** The type parameter `T` is inferred from the array argument. For `[1, 2, 3]`, `T = number`; for `["a", "b"]`, `T = string`. The return type `T | undefined` accurately reflects that the array may be empty. No explicit type argument is needed.

### Real-World Cases

- **Utility types**: `Array<T>`, `Promise<T>`, `Record<K, V>`, and `Partial<T>` are all built on generics.
- **API clients**: `fetchJson<T>(url: string): Promise<T>` returns a typed response without losing type information.
- **Data structures**: Stacks, queues, and linked lists are implemented generically to work with any element type.

---

## 2. Generic Functions

### Definitions

**Core Definition.** A **generic function** is a function that declares one or more **type parameters** and uses them in its parameter types, return type, or body.

**Technical Definition.** A generic function’s type is a **generic function type** — a function type parameterized over one or more type variables. The type parameters are listed in angle brackets before the parameter list. Type argument inference uses the types of the arguments at the call site to determine the type arguments, unless they are explicitly provided.

**Beginner-Friendly Explanation.** A generic function is **a function with a type placeholder**. You write `<T>` after the function name, and then use `T` wherever you want to say “whatever type the caller uses.” The function works for any type, but TypeScript still checks that the types line up.

### Purposes

- **To write reusable functions** that work over many types while preserving type relationships.
- **To enable type argument inference**, reducing the need for explicit type annotations at call sites.
- **To support higher-order functions** and callbacks with precise type flow.

### Syntax Rules and Structure

```typescript
function functionName<T, U>(param1: T, param2: U): ReturnType {
  // body
}
```

**Component breakdown:**

- `<T, U>` — the type parameter list; multiple parameters are separated by commas.
- `param1: T` — a parameter typed by the first type parameter.
- `param2: U` — a parameter typed by the second type parameter.
- `): ReturnType` — the return type, which may reference `T`, `U`, or both.

**Syntax rules.**

- Type parameters are declared before the parameter list.
- Type arguments may be explicit (`fn<number, string>(1, "a")`) or inferred from arguments.
- Constraints may be applied with `extends` (e.g., `<T extends object>`).
- Defaults may be provided (e.g., `<T = string>`).

**Constraints and limitations.**

- Unconstrained type parameters permit only operations that are valid for all types; accessing `.length` or other members requires a constraint.
- Inference may fail in complex positions; explicit type arguments resolve ambiguity.
- Generic function declarations with bodies that always throw infer `void` unless annotated `never`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic function with multiple type parameters

```typescript
// example_generic_function_pair.ts
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

#### Example 2: Generic function with a constraint

```typescript
// example_generic_function_constraint.ts
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(value: T): T {
  console.log(value.length);
  return value;
}

logLength("hello");          // Expected: 5
logLength([1, 2, 3]);        // Expected: 3
// logLength(42);            // Error: number does not satisfy HasLength
```

**Expected Output:**
```
5
3
```

**Why this output occurs.** The constraint `T extends HasLength` guarantees that every `T` has a `length` property, so `value.length` is legal. Strings and arrays satisfy `HasLength`; numbers do not, so the call with `42` is a compile-time error.

### Real-World Cases

- **Array utilities**: `map<T, U>(arr: T[], fn: (item: T) => U): U[]` transforms arrays with full type preservation.
- **Event emitters**: `emit<T>(event: string, payload: T)` types the payload per event.
- **API wrappers**: `request<T>(url: string): Promise<T>` types the response.

---

## 3. Generic Interfaces

### Definitions

**Core Definition.** A **generic interface** is an interface that declares one or more **type parameters**, allowing its members to reference those parameters.

**Technical Definition.** A generic interface describes a family of object types parameterized by one or more type variables. Type parameters may be placed on the **call signature** (making individual methods generic) or on the **interface itself** (making the whole type generic). The placement determines whether the type argument is fixed for the entire object or per-method invocation.

**Beginner-Friendly Explanation.** A generic interface is **a blueprint for objects that can hold or produce different types**. For example, a `Box<T>` interface describes an object with a `value: T` property — one blueprint, many concrete types (`Box<number>`, `Box<string>`).

### Purposes

- **To describe generic object shapes** without duplicating interface declarations for each type.
- **To support generic call signatures** and reusable function-object contracts.
- **To model containers, wrappers, and repositories** that work over arbitrary payload types.

### Syntax Rules and Structure

```typescript
interface InterfaceName<TypeParam> {
  property: TypeParam;
  method(param: TypeParam): TypeParam;
}
```

**Component breakdown:**

- `InterfaceName<TypeParam>` — the generic interface name and its type parameter list.
- `property: TypeParam` — a property whose type is the type parameter.
- `method(param: TypeParam): TypeParam` — a method using the type parameter.

**Syntax rules.**

- Type parameters are declared after the interface name.
- When using the interface, type arguments must be supplied (`Box<number>`) or inferred from context.
- Type parameters may have constraints (`<T extends object>`) and defaults (`<T = string>`).

**Constraints and limitations.**

- A generic interface that never uses its type parameter is discouraged.
- Interfaces may merge with other interfaces; merged declarations may introduce defaults for existing type parameters but must follow the default rules.
- Generic interfaces cannot be instantiated directly; they describe shapes only.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic container interface

```typescript
// example_generic_interface.ts
interface Box<T> {
  value: T;
}

const numberBox: Box<number> = { value: 42 };
const stringBox: Box<string> = { value: "hello" };

console.log(numberBox.value);   // Expected: 42
console.log(stringBox.value);   // Expected: hello
```

**Expected Output:**
```
42
hello
```

**Why this output occurs.** `Box<T>` is a generic interface. `Box<number>` requires `value: number`; `Box<string>` requires `value: string`. The interface describes the shape once, and the type argument specializes it.

#### Example 2: Type parameter on the call signature vs the interface

```typescript
// example_generic_interface_call_signature.ts
// Type parameter on the call signature: each call may use a different type.
interface GenericIdentityFn {
  <T>(arg: T): T;
}

// Type parameter on the interface: the type is fixed for the object.
interface GenericIdentityFnFixed<T> {
  (arg: T): T;
}

const identity: GenericIdentityFn = (arg) => arg;
const identityNumber: GenericIdentityFnFixed<number> = (arg) => arg;

console.log(identity("a"));         // Expected: a
console.log(identity(1));           // Expected: 1
console.log(identityNumber(2));     // Expected: 2
```

**Expected Output:**
```
a
1
2
```

**Why this output occurs.** `GenericIdentityFn` places `<T>` on the call signature, so each invocation infers its own `T`. `GenericIdentityFnFixed<number>` fixes `T = number`, so the function must accept and return `number`. This distinction is central to designing generic interfaces.

### Real-World Cases

- **Repositories**: `Repository<T>` with `get(id: string): Promise<T>` and `save(entity: T): Promise<void>`.
- **React props**: `Props<T>` interfaces for components that render generic data.
- **API responses**: `ApiResponse<T>` with `data: T` and `error?: string`.

---

## 4. Generic Classes

### Definitions

**Core Definition.** A **generic class** is a class that declares one or more **type parameters**, which may be used in property types, method signatures, and the constructor.

**Technical Definition.** A generic class has a type parameter list after the class name. Instantiating the class with type arguments produces a concrete class type whose members are specialized to those arguments. Generic classes support the same constraints and defaults as generic functions and interfaces.

**Beginner-Friendly Explanation.** A generic class is **a class that can work with different types**. A `Stack<T>` class can be a stack of numbers, strings, or anything else, but TypeScript tracks which type each instance holds.

### Purposes

- **To build reusable data structures** (stacks, queues, trees, caches) that work over arbitrary element types.
- **To enforce type consistency** between a class’s internal state and its public API.
- **To support generic factories and dependency injection** with typed constructors.

### Syntax Rules and Structure

```typescript
class ClassName<TypeParam> {
  property: TypeParam;
  constructor(property: TypeParam) {
    this.property = property;
  }
  method(): TypeParam {
    return this.property;
  }
}
```

**Component breakdown:**

- `ClassName<TypeParam>` — the class name and its type parameter list.
- `property: TypeParam` — an instance property typed by the parameter.
- `constructor(property: TypeParam)` — the constructor parameter.
- `method(): TypeParam` — a method returning the type parameter.

**Syntax rules.**

- Type parameters follow the class name.
- Type arguments are supplied when instantiating: `new ClassName<number>(42)`.
- Constraints (`<T extends object>`) and defaults (`<T = string>`) are supported.
- Generic classes may implement generic interfaces.

**Constraints and limitations.**

- A generic class that does not use its type parameter is pointless and discouraged.
- Static members cannot reference class type parameters.
- Generic classes cannot have generic `static` members.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic stack

```typescript
// example_generic_class_stack.ts
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop());   // Expected: 2

const stringStack = new Stack<string>();
stringStack.push("hello");
console.log(stringStack.pop());   // Expected: hello
```

**Expected Output:**
```
2
hello
```

**Why this output occurs.** `Stack<number>` ensures that only numbers can be pushed, and `pop()` returns `number | undefined`. `Stack<string>` enforces string consistency. The type parameter is fixed for each instance, preventing mixing of types within a stack.

#### Example 2: Generic class with a constraint

```typescript
// example_generic_class_constraint.ts
interface Identifiable {
  id: string;
}

class Repository<T extends Identifiable> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  findById(id: string): T | undefined {
    return this.items.find((item) => item.id === id);
  }
}

const userRepo = new Repository<{ id: string; name: string }>();
userRepo.add({ id: "u1", name: "Ada" });
console.log(userRepo.findById("u1")?.name);   // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The constraint `T extends Identifiable` guarantees that every `T` has an `id: string` property, making `findById`’s comparison legal. The type argument `{ id: string; name: string }` satisfies the constraint.

### Real-World Cases

- **Data structures**: `Queue<T>`, `LinkedList<T>`, `BinaryTree<T>`.
- **Repositories**: `Repository<T extends Entity>` for database access.
- **Caches**: `Cache<K, V>` with generic key and value types.
- **Event buses**: `EventEmitter<EventMap>` where `EventMap` maps event names to payload types.

---

## 5. Generic Type Aliases

### Definitions

**Core Definition.** A **generic type alias** is a type alias that declares one or more **type parameters**, allowing the alias to produce different types depending on the arguments supplied.

**Technical Definition.** A generic type alias introduces a **type constructor** — a named type-level function from type arguments to a type. It is especially useful for unions, intersections, tuples, and conditional types that cannot be expressed as interfaces. Type parameters follow the alias name and support constraints and defaults.

**Beginner-Friendly Explanation.** A generic type alias is **a nickname for a type that depends on other types**. For example, `type Pair<T> = [T, T]` defines a tuple of two values of the same type. `Pair<number>` is `[number, number]`, and `Pair<string>` is `[string, string]`.

### Purposes

- **To name reusable type-level patterns** such as unions, intersections, and tuples.
- **To build discriminated unions** that are parameterized over payload types.
- **To support generic utility types** and type-level computations.
- **To avoid duplicating type declarations** across similar shapes.

### Syntax Rules and Structure

```typescript
type AliasName<TypeParam> = TypeExpression;
```

**Component breakdown:**

- `AliasName<TypeParam>` — the alias name and its type parameter list.
- `TypeExpression` — any type expression that may reference the type parameter.

**Syntax rules.**

- Type parameters follow the alias name.
- Constraints use `extends`; defaults use `=`.
- Generic aliases can be recursive (e.g., JSON types) and can reference other generic aliases.
- Generic aliases cannot be declaration-merged, unlike interfaces.

**Constraints and limitations.**

- Recursive generic aliases must terminate; infinite recursion is a compile-time error.
- Type aliases are erased and cannot be extended or implemented.
- Generic aliases with unused type parameters are discouraged.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic tuple alias

```typescript
// example_generic_alias_tuple.ts
type Pair<T> = [T, T];

const coordinates: Pair<number> = [10, 20];
const names: Pair<string> = ["Ada", "Grace"];

console.log(coordinates[0] + coordinates[1]);   // Expected: 30
console.log(names.join(" & "));                  // Expected: Ada & Grace
```

**Expected Output:**
```
30
Ada & Grace
```

**Why this output occurs.** `Pair<T>` is a tuple type parameterized by `T`. `Pair<number>` enforces that both elements are numbers, enabling numeric addition. `Pair<string>` enforces string elements, enabling `join`.

#### Example 2: Generic discriminated union

```typescript
// example_generic_alias_union.ts
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

function unwrap<T>(result: Result<T>): T {
  if (result.success) {
    return result.data;   // narrowed to success branch
  }
  throw new Error(result.error);
}

console.log(unwrap({ success: true, data: 42 }));   // Expected: 42
```

**Expected Output:**
```
42
```

**Why this output occurs.** `Result<T>` is a generic discriminated union. The `success` discriminant narrows the union inside `unwrap`, allowing safe access to `data` in the success branch and `error` in the failure branch. The type argument `T` flows through the entire union.

### Real-World Cases

- **API responses**: `type ApiResult<T> = { ok: true; data: T } | { ok: false; error: string }`.
- **Form state**: `type FieldState<T> = { value: T; touched: boolean }`.
- **JSON types**: Recursive generic aliases model JSON values.
- **Function types**: `type Mapper<T, U> = (input: T) => U`.

---

## 6. Type Parameters

### Definitions

**Core Definition.** A **type parameter** is a **type-level variable** declared in a generic declaration, standing in for a type that will be supplied as a type argument when the generic is used.

**Technical Definition.** Type parameters are declared in angle brackets (`<T>`). They may be **constrained** with `extends` (restricting the set of permissible type arguments), **defaulted** with `=` (providing a fallback when no type argument is supplied), and **annotated for variance** with `in` and `out` (TypeScript 4.7+) to document intended variance. A type parameter is **in scope** throughout the declaration’s body and signature.

**Beginner-Friendly Explanation.** A type parameter is **a placeholder for a type**. When you write `function identity<T>(arg: T)`, `T` is the placeholder. When you call `identity(42)`, TypeScript replaces `T` with `number`. Constraints let you say “T must have a `length` property,” and defaults let you say “use `string` if no type is given.”

### Purposes

- **To declare a type variable** that can be used throughout a generic declaration.
- **To constrain the set of acceptable type arguments** with `extends`.
- **To provide a sensible fallback** with defaults, improving usability for common cases.
- **To document variance** with `in`/`out` annotations for correctness and performance.

### Syntax Rules and Structure

```typescript
<T>                          // unconstrained type parameter
<T extends Constraint>       // constrained type parameter
<T = DefaultType>            // defaulted type parameter
<T extends Constraint = Default>  // constrained and defaulted
<in T>                       // contravariant annotation
<out T>                      // covariant annotation
```

**Component breakdown:**

- `T` — the type parameter name (conventionally single uppercase letters or descriptive names like `TKey`, `TValue`).
- `extends Constraint` — restricts `T` to types assignable to `Constraint`.
- `= Default` — supplies a fallback type when no type argument is provided.
- `in` / `out` — variance annotations (optional, advanced).

**Syntax rules.**

- Required type parameters must not follow optional (defaulted) type parameters.
- Default types must satisfy the parameter’s constraint, if any.
- Constraints may reference other type parameters (e.g., `<T, K extends keyof T>`).
- Variance annotations are only consulted in instantiation-based comparisons; they do not change structural behavior.

**Constraints and limitations.**

- Unconstrained type parameters permit only operations valid for all types.
- Defaults should only be used when the type is meaningfully usable without specialization; otherwise they create a false sense of optionality.
- Variance annotations should not be used to “force” variance contrary to structural behavior; TypeScript infers variance automatically in most cases.

### Annotated Complete Step by Step Code Examples

#### Example 1: Constrained type parameter with `keyof`

```typescript
// example_type_parameter_keyof.ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "Ada", age: 36 };
const name = getProperty(person, "name");   // string
const age = getProperty(person, "age");     // number

console.log(name);   // Expected: Ada
console.log(age);    // Expected: 36
```

**Expected Output:**
```
Ada
36
```

**Why this output occurs.** The constraint `K extends keyof T` ensures that `key` is a valid property name of `obj`. The return type `T[K]` is the **indexed access type** — the type of the property `K` on `T`. For `"name"`, `T[K]` is `string`; for `"age"`, it is `number`.

#### Example 2: Default type parameter

```typescript
// example_type_parameter_default.ts
interface Container<T = string> {
  value: T;
}

const defaultContainer: Container = { value: "hello" };
// T defaults to string
const numberContainer: Container<number> = { value: 42 };

console.log(defaultContainer.value);   // Expected: hello
console.log(numberContainer.value);    // Expected: 42
```

**Expected Output:**
```
hello
42
```

**Why this output occurs.** `Container` declares `T = string` as a default. When no type argument is supplied, `T` resolves to `string`. When `Container<number>` is used, `T` is `number`, overriding the default.

### Real-World Cases

- **Utility functions**: `getProperty<T, K extends keyof T>` is a pattern for type-safe property access.
- **Configuration types**: `Options<T = {}>` allows callers to omit type arguments for simple cases.
- **Variance-sensitive types**: `Producer<out T>` and `Consumer<in T>` document covariance and contravariance in libraries.

---

## 7. Type Arguments

### Definitions

**Core Definition.** A **type argument** is the **concrete type** supplied to a generic declaration in place of a type parameter, either explicitly in angle brackets or implicitly through type argument inference.

**Technical Definition.** Type arguments instantiate a generic type constructor, producing a concrete type. They are checked against the type parameter’s constraints. When omitted, TypeScript infers them from the types of arguments, contextual types, and return positions. Explicit type arguments override inference and are required when inference cannot determine a unique type.

**Beginner-Friendly Explanation.** A type argument is **the actual type you give to a generic**. In `identity<number>(42)`, `number` is the type argument. In `identity(42)`, TypeScript infers `number` automatically. Type arguments fill in the placeholders.

### Purposes

- **To specialize a generic declaration** to a concrete type.
- **To override inference** when the inferred type is not what you intend.
- **To satisfy constraints** and enable type checking of the instantiation.
- **To provide explicit type information** in complex cases where inference is ambiguous.

### Syntax Rules and Structure

```typescript
genericFunction<TypeArgument>(arguments);
new GenericClass<TypeArgument>(constructorArgs);
const value: GenericInterface<TypeArgument> = ...;
```

**Component breakdown:**

- `genericFunction<TypeArgument>` — explicit type argument on a function call.
- `new GenericClass<TypeArgument>` — explicit type argument on a class instantiation.
- `GenericInterface<TypeArgument>` — type argument in a type annotation.

**Syntax rules.**

- Type arguments are supplied in angle brackets after the generic name.
- When explicit type arguments are provided, inference is bypassed for those parameters.
- Partial type argument lists are allowed when remaining parameters have defaults.
- Type arguments must satisfy the corresponding type parameter’s constraints.

**Constraints and limitations.**

- Providing the wrong number of type arguments is a compile-time error.
- Explicit type arguments can conflict with argument types, causing errors.
- Type arguments are erased at runtime; they have no runtime representation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit vs inferred type arguments

```typescript
// example_type_arguments.ts
function createArray<T>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

// Explicit type argument.
const strings = createArray<string>(3, "x");   // string[]

// Inferred type argument.
const numbers = createArray(3, 0);             // number[]

console.log(strings);   // Expected: ["x", "x", "x"]
console.log(numbers);   // Expected: [0, 0, 0]
```

**Expected Output:**
```
["x", "x", "x"]
[0, 0, 0]
```

**Why this output occurs.** `createArray<string>` explicitly sets `T = string`. `createArray(3, 0)` infers `T = number` from `0`. Both produce arrays of the correct element type.

#### Example 2: Explicit type argument resolving ambiguity

```typescript
// example_type_arguments_ambiguity.ts
function combine<T>(a: T, b: T): T[] {
  return [a, b];
}

// Inference may produce a union when types differ.
const mixed = combine(1, "two");   // (string | number)[]

// Explicit type argument forces a common type.
const asStrings = combine<string>("1", "two");   // string[]

console.log(mixed);       // Expected: [1, "two"]
console.log(asStrings);   // Expected: ["1", "two"]
```

**Expected Output:**
```
[1, "two"]
["1", "two"]
```

**Why this output occurs.** With `combine(1, "two")`, inference produces `T = string | number`. Explicit `combine<string>` forces `T = string`, and the number `1` must be assignable to `string` — which it is not, so this would actually be a type error unless a string is passed. The example illustrates how explicit type arguments control inference.

### Real-World Cases

- **React hooks**: `useState<number>(0)` explicitly types state when inference from the initial value is insufficient.
- **API clients**: `fetchJson<User>(url)` explicitly types the response payload.
- **Generic containers**: `new Map<string, number>()` supplies both key and value type arguments.

---

## 8. Generic Best Practices

### Definitions

**Core Definition.** **Generic best practices** are guidelines for designing and using generics in ways that maximize **reusability, type safety, clarity, and performance** while avoiding common pitfalls such as unused type parameters, unnecessary defaults, and unconstrained generics.

**Technical Definition.** Best practices include: using **as few type parameters as possible**; ensuring each type parameter **appears at least twice** in the signature; **constraining generics** when specific members are accessed; avoiding **return-type-only generics**; providing **defaults only when meaningful**; and using **variance annotations** sparingly and only when they match structural behavior.

**Beginner-Friendly Explanation.** Best practices help you write generics that are **actually useful and not confusing**. Use generics when you need to preserve a type relationship; don’t use them just to look advanced. If a type parameter appears only once, you probably don’t need it. Constrain your generics when you need to use specific properties. Avoid defaults unless they make sense.

### Purposes

- **To maximize type safety** by ensuring generics preserve meaningful type relationships.
- **To minimize complexity** by avoiding unnecessary type parameters and defaults.
- **To improve API usability** by constraining generics appropriately and providing defaults only when useful.
- **To avoid common generics pitfalls** such as unused type parameters and return-type-only generics.

### Syntax Rules and Structure

**Key best-practice rules:**

1. **Use fewer type parameters**: prefer `function log<T>(value: T)` over unnecessary multi-parameter forms.
2. **Ensure each type parameter appears at least twice**: a type parameter used only once is likely unnecessary.
3. **Constrain generics when accessing members**: use `<T extends HasLength>` before calling `.length`.
4. **Avoid return-type-only generics**: do not create APIs where the type parameter appears only in the return type; callers cannot infer it, and explicit specification is required.
5. **Use defaults only when the type is meaningful without specialization**: otherwise, defaults create a false sense of optionality.
6. **Use variance annotations sparingly**: TypeScript infers variance automatically; annotations are only for correctness documentation and rare performance optimization.
7. **Use descriptive type parameter names** when the meaning is not obvious (`TKey`, `TValue` rather than `T`, `U`).

**Constraints and limitations.**

- Over-constraining generics reduces reusability; find the minimal constraint needed.
- Defaults that are not meaningful create confusing APIs.
- Variance annotations that do not match structural behavior cause unpredictable behavior.

### Annotated Complete Step by Step Code Examples

#### Example 1: Fewer type parameters and ensuring each appears twice

```typescript
// example_best_fewer_params.ts
// Recommended: one type parameter, used twice.
function wrap<T>(value: T): { value: T } {
  return { value };
}

// Discouraged: unnecessary second type parameter.
// function wrapBad<T, U>(value: T): { value: U } { ... }
// U appears only once and cannot be inferred.

console.log(wrap(42).value);      // Expected: 42
console.log(wrap("hello").value); // Expected: hello
```

**Expected Output:**
```
42
hello
```

**Why this output occurs.** `T` appears in both the parameter and the return type, preserving the type relationship. A second type parameter `U` appearing only in the return type cannot be inferred and would require explicit specification at every call site.

#### Example 2: Constraint before member access

```typescript
// example_best_constraint.ts
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}

console.log(longest("hello", "hi"));          // Expected: hello
console.log(longest([1, 2, 3], [4, 5]));      // Expected: [1, 2, 3]
// longest(1, 2);                             // Error: number has no length
```

**Expected Output:**
```
hello
[1, 2, 3]
```

**Why this output occurs.** The constraint `T extends { length: number }` guarantees that `.length` is available on both arguments. Strings and arrays satisfy the constraint; numbers do not. This is safer than leaving `T` unconstrained and casting to `any`.

#### Example 3: Defaults only when meaningful

```typescript
// example_best_defaults.ts
// Recommended: default is meaningful (string is a sensible no-specialization type).
interface Formatter<T = string> {
  format(value: T): string;
}

// Discouraged: default creates a false sense of optionality.
// interface Repository<T = never> { ... }
// T is required for the repository to be useful.

const stringFormatter: Formatter = {
  format: (v) => v.toUpperCase(),
};

console.log(stringFormatter.format("hello"));   // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** `T = string` is a meaningful default because a string formatter is a valid, useful configuration without specialization. A default like `never` would not be meaningful and would mislead callers into thinking the type is optional when it is not.

### Real-World Cases

- **Library design**: Constrained generics with meaningful defaults provide safe, ergonomic APIs.
- **Utility types**: `Pick<T, K extends keyof T>` and `Record<K extends keyof any, T>` use constraints to ensure correctness.
- **Return-type-only generics**: Explicit type arguments are required when the return type cannot be inferred, such as `useState<number>(0)`.

---

## References

- TypeScript Handbook — Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook — Creating Types from Types - https://www.typescriptlang.org/docs/handbook/2/types-from-types.html
- TypeScript Handbook — Do’s and Don’ts (Generics) - https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
- TypeScript Release Notes 2.6 — Strict Function Types and Variance - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-6.html
- TypeScript 4.7 Release Notes — Optional Variance Annotations - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html
- Google TypeScript Style Guide — Return type only generics - https://google.github.io/styleguide/tsguide.html
- Learning TypeScript — Generics - https://www.learningtypescript.com/generics
- TypeScript Generics Reference — Constraints, Defaults, NoInfer - https://cdn.jsdelivr.net/npm/typescript-generics-reference