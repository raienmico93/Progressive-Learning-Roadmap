# TypeScript Type System Fundamentals — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** The TypeScript type system is a **gradual**, **structural**, **erased** type system layered over JavaScript. It permits both static and dynamic typing within the same program, relates types by their member structure rather than by explicit declaration, and removes all type information during compilation so that emitted JavaScript carries no runtime type metadata.

**Technical Definition.** TypeScript’s type system performs **compile-time type checking** using a set of type-relation rules — principally **structural subtyping** — and supports **optional type annotations** that the compiler uses for inference and checking. After checking completes, the compiler **erases** all type annotations and type-only constructs, emitting plain JavaScript that is semantically equivalent to the source minus types. The system is **deliberately unsound** in specific, documented places to preserve JavaScript idioms and ergonomics.

**Beginner-Friendly Explanation.** TypeScript is JavaScript with a **type checker riding alongside**. You can write JavaScript exactly as before, and TypeScript will try to figure out what types your values have. When it can, it tells you early — before you run the code — if something looks wrong. But once your code is compiled to JavaScript, the types disappear completely. They are a **development-time safety net**, not a runtime enforcement mechanism.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Gradual** | TypeScript allows optional typing; unannotated JavaScript is valid TypeScript and is checked with inference and `any` where necessary. |
| **Structural** | Type compatibility is determined by members, not by declared names or inheritance chains. |
| **Erased** | All type annotations, interfaces, type aliases, and type-only imports are removed during compilation. |
| **Unsound in controlled ways** | TypeScript explicitly permits operations that cannot be proven safe at compile time, such as bivariant method parameter checking and `any` assignability. |
| **Context-sensitive inference** | Types are inferred from initializers, return expressions, contextual positions, and best-common-type algorithms. |
| **Nominal escape hatches** | Classes and `private`/`protected` members introduce nominal-like behavior into an otherwise structural system. |
| **Runtime cost: none** | Because types are erased, they do not affect runtime performance or memory footprint. |

---

### Prerequisites

To follow this cheat sheet effectively, you should be comfortable with:

- **JavaScript fundamentals**: variables (`let`, `const`), functions, objects, arrays, classes, modules.
- **Basic programming concepts**: values, expressions, statements, scope, closures.
- **A code editor with TypeScript support** (e.g., VS Code) or access to the **TypeScript Playground**.
- **Node.js and npm** (optional but recommended) to run the TypeScript compiler (`tsc`) locally.

No prior experience with type theory is assumed. Type-theoretic terms are introduced and explained as they appear.

---

### Related Programming Areas

- **Gradual typing research** (e.g., Typed Racket, Python type hints, Flow).
- **Structural type systems** (e.g., Go interfaces, OCaml object types).
- **Nominal type systems** (e.g., Java, C#, C++).
- **Duck typing** in dynamically typed languages (e.g., Python, Ruby).
- **Type erasure** in compiled languages (e.g., Java generics, C# reification contrast).
- **Language server protocols and developer tooling** (TypeScript’s type system powers IntelliSense, refactoring, and error reporting).

---

### Core Concepts / Features

The following eleven concepts constitute the core of TypeScript’s type system fundamentals. Each is presented with the same uniform structure: definitions, purposes, syntax rules, annotated executable code examples with outputs, and real-world cases.

---

## 1. Type System

### Definitions

**Core Definition.** A **type system** is a set of rules that assigns **types** to program constructs — variables, expressions, functions, modules — and defines when one type is compatible with another. It governs which operations are legal and which programs are well-typed.

**Technical Definition.** Formally, a type system consists of a collection of **types**, a **typing relation** \(\Gamma \vdash e : \tau\) (expression \(e\) has type \(\tau\) under context \(\Gamma\)), and a set of **subtyping** and **compatibility** rules. TypeScript’s type system includes primitive types, object types, union and intersection types, literal types, generic types, and special types (`any`, `unknown`, `never`, `void`).

**Beginner-Friendly Explanation.** Think of a type system as a **labeling and rulebook system** for your code. Every value gets a label (its type). The rulebook says which labels can be used where. If you try to use a value in a way that violates its label, the type system flags it.

### Purposes

- **To detect type errors early**, before the program runs.
- **To document the intended shape and behavior of code** through machine-checkable annotations.
- **To enable powerful tooling** such as autocompletion, go-to-definition, and safe refactoring.
- **To provide a foundation for reasoning about program correctness** within the limits of the system’s soundness guarantees.

### Syntax Rules and Structure

**General syntax.** TypeScript’s type system has no single “syntax” — it is a collection of type-level and value-level constructs. The core syntactic categories are:

1. **Type annotations**: `: Type`
2. **Type aliases**: `type Name = Type`
3. **Interfaces**: `interface Name { ... }`
4. **Type operators**: `|` (union), `&` (intersection), `keyof`, `typeof`, `infer`, conditional types, mapped types.
5. **Generic type parameters**: `<T>`, `<T extends Constraint>`.

**Component breakdown (representative annotation syntax):**

```
let variableName: TypeName = value;
```

- `let` / `const` / `var` — variable declaration keyword.
- `variableName` — identifier being declared.
- `:` — **type annotation separator** (not an operator; it introduces the type expression).
- `TypeName` — a type expression: a primitive (`string`, `number`, `boolean`), an object type, a union, a generic instantiation, etc.
- `= value` — optional initializer; its type contributes to inference and checking.

**Syntax rules.**

- Type annotations appear after the identifier in variable declarations, after parameter names in function signatures, and after `)` before `=>` or `{` for return types.
- Type aliases and interfaces introduce **type-only names** that are erased at compilation.
- Generic parameters must be declared before use and may have constraints via `extends`.

**Constraints and limitations.**

- TypeScript’s type system is **not fully sound**; it permits some unsafe operations by design.
- Types have **no runtime representation**; you cannot `instanceof` an interface or query a type alias at runtime.
- Some type-level computations can become expensive or hit recursion limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: Primitive and object type annotations

```typescript
// example1.ts
// Step 1: Declare a variable with an explicit primitive type annotation.
let userName: string = "Alice";        // ': string' annotates userName as string

// Step 2: Declare a variable with an inferred type (no annotation).
let userAge = 30;                        // TypeScript infers number

// Step 3: Declare an object type inline.
let user: { name: string; age: number } = {
  name: userName,
  age: userAge,
};

// Step 4: Attempt an invalid assignment to observe static checking.
// user.age = "thirty";                  // Uncommenting causes a compile-time error

console.log(user.name);                  // Expected: Alice
console.log(user.age);                   // Expected: 30
```

**Setup.** Save as `example1.ts`. Run `npx tsc example1.ts --strict` then `node example1.js`.

**Expected Output:**
```
Alice
30
```

**Why this output occurs.** The compiler checks that `user.age` is a `number` and that `user.name` is a `string`. The attempted assignment `"thirty"` to `age` would fail type checking because `string` is not assignable to `number`. At runtime, the types are erased; only the JavaScript assignments and `console.log` calls remain.

#### Example 2: Interface and type alias

```typescript
// example2.ts
// Step 1: Define an interface describing an object shape.
interface Product {
  id: number;
  name: string;
}

// Step 2: Define a type alias for a union of string literals.
type Status = "active" | "inactive";

// Step 3: Create a value conforming to the interface.
const item: Product = { id: 1, name: "Keyboard" };

// Step 4: Declare a variable of the alias type.
let productStatus: Status = "active";

// Step 5: Log the values.
console.log(`${item.name} is ${productStatus}`);
// Expected: Keyboard is active
```

**Expected Output:**
```
Keyboard is active
```

**Why this output occurs.** `Product` and `Status` are type-level constructs. The object literal is checked against `Product`, and `"active"` is checked against the union `"active" | "inactive"`. Both checks pass. During compilation, `interface` and `type` declarations are erased entirely.

### Real-World Cases

- **API contract modeling**: Interfaces and type aliases describe the shape of JSON payloads, enabling editors to autocomplete response fields and flag mismatches before runtime.
- **Domain modeling with literal unions**: `type OrderState = "pending" | "shipped" | "delivered"` prevents invalid state strings at compile time.
- **Library authoring**: Declaration files (`.d.ts`) expose type information to consumers without shipping implementation code.

---

## 2. Static Typing

### Definitions

**Core Definition.** **Static typing** is the property of a language or type system whereby type checking is performed **before program execution**, typically at compile time, using the declared or inferred types of expressions.

**Technical Definition.** In a statically typed system, the typing judgment \(\Gamma \vdash e : \tau\) is established during compilation. Type errors are detected as part of the translation from source to target language. TypeScript performs static checking via the `tsc` compiler and editor language services, but because types are erased, the checks do not persist into the emitted JavaScript.

**Beginner-Friendly Explanation.** Static typing means the type checker **reads your code before you run it** and tells you about type mistakes. It is like a proofreader who checks your essay before you submit it, rather than discovering problems only when a reader tries to understand it.

### Purposes

- **To catch type mismatches at development time**, reducing runtime failures.
- **To make refactoring safer** by identifying all affected locations when a type changes.
- **To serve as executable documentation** of intended interfaces.
- **To enable richer IDE support** based on known types.

### Syntax Rules and Structure

Static typing in TypeScript is enabled through **type annotations**, **inference**, and compiler flags. The primary syntactic vehicle is the annotation `: Type`. Compiler flags — especially `--strict`, `--noImplicitAny`, and `--strictNullChecks` — control the strictness of static checking.

**Component breakdown of a strict-mode annotation:**

```
function greet(name: string): string {
  return `Hello, ${name}`;
}
```

- `name: string` — parameter type annotation; `name` must receive a `string`.
- `): string` — return type annotation; the function must return a `string`.
- With `--strict`, omitting these annotations may trigger implicit-`any` errors in positions where inference cannot determine a type.

**Syntax rules.**

- Annotations are optional in TypeScript; static checking still occurs through inference.
- `--strict` enables a family of strictness flags, including `noImplicitAny`, `strictNullChecks`, `strictFunctionTypes`, and `strictPropertyInitialization`.
- Type assertions (`as T`) override static checking locally but do **not** perform runtime checks.

**Constraints and limitations.**

- Static checking cannot catch all runtime errors; TypeScript is unsound in specific ways.
- Types are erased, so no runtime type validation occurs automatically.
- Dynamic constructs — `eval`, `Function` constructors, dynamic property access with arbitrary strings — may escape static checking.

### Annotated Complete Step by Step Code Examples

#### Example 1: Static error detection

```typescript
// example_static.ts
// Step 1: Define a function with explicit parameter and return types.
function double(n: number): number {
  return n * 2;
}

// Step 2: Call the function correctly.
const result = double(5);
console.log(result); // Expected: 10

// Step 3: Uncomment to observe a static type error.
// const bad = double("five");
// Compiler error: Argument of type 'string' is not assignable to parameter of type 'number'.
```

**Setup.** Compile with `npx tsc example_static.ts --strict`. The compiler reports an error on the commented line if uncommented; the emitted JavaScript still runs the valid call.

**Expected Output (with invalid line commented):**
```
10
```

**Why this output occurs.** The compiler statically verifies that `5` is assignable to `number`. The string argument would fail the static check because `string` is not assignable to `number`. At runtime, the check has already disappeared; only the multiplication and logging remain.

#### Example 2: `strictNullChecks` preventing null errors

```typescript
// example_strict_null.ts
// Step 1: Enable strictNullChecks via compiler flag.
// Run: npx tsc example_strict_null.ts --strict

function getLength(s: string): number {
  return s.length;
}

// Step 2: This call is statically safe.
console.log(getLength("hello")); // Expected: 5

// Step 3: Uncomment to see a strictNullChecks error.
// console.log(getLength(null));
// Error: Argument of type 'null' is not assignable to parameter of type 'string'.
```

**Expected Output:**
```
5
```

**Why this output occurs.** With `strictNullChecks`, `null` and `undefined` are not assignable to `string`. The compiler flags the unsafe call before runtime. This is a **static** guarantee that the function will never receive `null` through that call site.

### Real-World Cases

- **CI/CD pipelines**: `tsc --noEmit --strict` is commonly run in continuous integration to block merges that introduce type errors.
- **Large codebase refactoring**: Renaming a field in an interface triggers static errors at every usage site that still references the old field name.
- **Editor feedback**: VS Code and other editors use the TypeScript language service to underline type errors as you type, without requiring a full build.

---

## 3. Dynamic Typing

### Definitions

**Core Definition.** **Dynamic typing** is the property whereby types are associated with **runtime values** rather than with compile-time expressions, and type checking occurs **during execution**.

**Technical Definition.** In a dynamically typed language, the type of a variable is the type of the value it currently holds, and operations are checked at runtime against the actual value’s type. JavaScript is dynamically typed; TypeScript layers optional static typing over JavaScript but **does not remove dynamic typing from the runtime semantics**.

**Beginner-Friendly Explanation.** Dynamic typing means the type checker **waits until the program is running** to decide whether an operation is valid. A variable can hold a number at one moment and a string the next. The language checks the value’s type at the moment of use.

### Purposes

- **To allow flexible, expressive code** without upfront type declarations.
- **To support rapid prototyping** where types may be unclear or changing.
- **To interoperate with untyped JavaScript libraries and dynamic patterns** such as metaprogramming.

### Syntax Rules and Structure

Dynamic typing in TypeScript is expressed through:

- **`any`**: an escape hatch type that disables static checking for a value.
- **Unannotated variables in non-strict positions**: inference may produce `any` when no information is available.
- **Runtime constructs**: `typeof`, `instanceof`, property checks, and user-defined type guards can be used to perform dynamic checks.

**Component breakdown of `any`-based dynamic behavior:**

```
let value: any = 42;
value = "now a string";
value.nonExistentMethod();
```

- `: any` — opts out of static checking for `value`.
- Reassignment to any type is permitted.
- Calling a non-existent method is not a static error because `any` suppresses checking.

**Syntax rules.**

- `any` is assignable to and from every type.
- Under `--noImplicitAny`, unannotated positions that would otherwise become `any` produce errors.
- `unknown` is the type-safe counterpart of `any`: it accepts any value but requires narrowing before use.

**Constraints and limitations.**

- `any` disables type checking for the affected value and propagates outward.
- Dynamic checks at runtime have a performance cost.
- Overuse of `any` defeats the purpose of TypeScript; it is **deprecated as a default choice** in strict codebases.

### Annotated Complete Step by Step Code Examples

#### Example 1: `any` and dynamic reassignment

```typescript
// example_dynamic.ts
// Step 1: Declare a value of type any.
let dynamicValue: any = 10;

// Step 2: Reassign to a different type — allowed with any.
dynamicValue = "hello";

// Step 3: Access a property not known statically.
console.log(dynamicValue.toUpperCase()); // Expected: HELLO

// Step 4: Assign to a strictly typed variable — allowed because source is any.
let strictString: string = dynamicValue;
console.log(strictString); // Expected: hello
```

**Expected Output:**
```
HELLO
hello
```

**Why this output occurs.** `any` suppresses static checking, so the reassignment and property access are not flagged. The emitted JavaScript retains the dynamic operations, and at runtime `toUpperCase` exists on the string value. The assignment to `strictString` is also allowed because `any` is assignable to `string` — a deliberate unsound escape hatch.

#### Example 2: `unknown` as the safe alternative

```typescript
// example_unknown.ts
// Step 1: Declare a value of type unknown.
let safeValue: unknown = "world";

// Step 2: Uncommenting the next line is a compile-time error.
// console.log(safeValue.toUpperCase());
// Error: Object is of type 'unknown'.

// Step 3: Narrow with a typeof check.
if (typeof safeValue === "string") {
  console.log(safeValue.toUpperCase()); // Expected: WORLD
}
```

**Expected Output:**
```
WORLD
```

**Why this output occurs.** `unknown` accepts any value but requires the programmer to narrow it before use. The `typeof` check creates a type guard that narrows `safeValue` to `string` inside the `if` block. This preserves dynamic flexibility while maintaining static safety.

### Real-World Cases

- **Migrating JavaScript to TypeScript**: `any` is temporarily used for unported modules, then progressively replaced with precise types.
- **Interfacing with untyped third-party libraries**: `any` or `unknown` bridges the gap until type definitions are available.
- **Dynamic configuration objects**: values parsed from JSON or environment variables may be typed as `unknown` and narrowed with runtime validators.

---

## 4. Structural Typing

### Definitions

**Core Definition.** **Structural typing** is a type system discipline in which type compatibility is determined by the **members (properties, methods, signatures)** a type possesses, rather than by its name or explicit declaration of inheritance.

**Technical Definition.** In a structural type system, a type \(S\) is assignable to a type \(T\) if for every member required by \(T\), \(S\) provides a member with a compatible type. This is the basis of TypeScript’s type compatibility rules: “\(x\) is compatible with \(y\) if \(y\) has at least the same members as \(x\)”. The comparison proceeds recursively through member types.

**Beginner-Friendly Explanation.** Structural typing asks: **“Does this value have the right shape?”** It does not care what the type is called or who declared it. If an object has a `name` string and a `speak()` method, it can be used wherever a “thing with a `name` and a `speak()`” is expected — even if the object never said it implements that interface.

### Purposes

- **To align with JavaScript’s object-literal and duck-typing idioms** without requiring explicit `implements` declarations.
- **To reduce boilerplate** by allowing compatible types to be used interchangeably without inheritance relationships.
- **To model loosely coupled, interface-driven designs** where shape matters more than nominal identity.

### Syntax Rules and Structure

Structural typing is not a syntax you write directly; it is the **rule** by which the compiler relates types. You express types using interfaces, type aliases, object types, and classes, and the compiler applies structural compatibility.

**General syntax (interface definition):**

```
interface InterfaceName {
  propertyName: Type;
  methodName(param: Type): ReturnType;
}
```

**Component breakdown:**

- `interface` — introduces a named object type.
- `InterfaceName` — the type name (nominal label, but **not** used for compatibility).
- `propertyName: Type` — a required property and its type.
- `methodName(param: Type): ReturnType` — a required method signature.

**Syntax rules.**

- Structural compatibility is **recursive**: nested object types are compared member by member.
- Extra members in the source type are permitted; only members required by the target type are checked.
- Functions are compared structurally: parameter types and return types must be compatible, with special rules for parameter bivariance and arity.
- Object literals assigned directly to a typed variable are subject to **excess property checks**, a stricter variant of structural checking.

**Constraints and limitations.**

- Structural typing can produce unexpected compatibility: two unrelated types with identical shapes are interchangeable, which may be undesirable for domain modeling.
- Private and protected members introduce nominal-like behavior: two classes with private members are compatible only if the private member originates from the same declaration.
- Structural comparisons can become costly on very large types.

### Annotated Complete Step by Step Code Examples

#### Example 1: Structural compatibility across unrelated classes

```typescript
// example_structural.ts
// Step 1: Define an interface.
interface Named {
  name: string;
}

// Step 2: Define a class that does NOT implement Named.
class Person {
  constructor(public name: string) {}
}

// Step 3: Assign a Person instance to a Named-typed variable.
const named: Named = new Person("Ada");

// Step 4: Log the result.
console.log(named.name); // Expected: Ada
```

**Setup.** Compile with `npx tsc example_structural.ts --strict`.

**Expected Output:**
```
Ada
```

**Why this output occurs.** `Person` has a public `name: string` property. `Named` requires `name: string`. Because TypeScript is structural, the assignment succeeds even though `Person` does not explicitly `implement Named`. In a nominal system like Java or C#, this would be an error.

#### Example 2: Functions compared structurally

```typescript
// example_structural_functions.ts
// Step 1: Define a function type.
type StringLogger = (message: string) => void;

// Step 2: Define a compatible function.
const logger: StringLogger = (msg) => {
  console.log(msg.toUpperCase());
};

// Step 3: Call it.
logger("structural"); // Expected: STRUCTURAL

// Step 4: A function with fewer parameters is compatible.
const shortLogger: StringLogger = () => {
  console.log("ignored parameter");
};
shortLogger("anything"); // Expected: ignored parameter
```

**Expected Output:**
```
STRUCTURAL
ignored parameter
```

**Why this output occurs.** Function compatibility in TypeScript is structural: the parameter names are irrelevant, and a function that accepts fewer parameters is assignable to a function type that expects more, because it simply ignores the extra argument. This is a deliberate design choice for JavaScript callback patterns.

### Real-World Cases

- **React props**: Components are typed by the shape of their props; a component accepts any props object with the required fields, regardless of how the object was created.
- **Node.js middleware**: Middleware functions are compatible if they accept the required request/response shapes, enabling composition without inheritance.
- **Cross-library interoperability**: A type from library A is usable where library B expects a structurally identical type, without adapters.

---

## 5. Nominal Typing

### Definitions

**Core Definition.** **Nominal typing** is a type system discipline in which type compatibility is determined by **explicit declarations of type identity** — names, inheritance relationships, or `implements` clauses. Two types are compatible only if their declarations establish a relationship.

**Technical Definition.** In a nominal system, \(S\) is a subtype of \(T\) only if \(S\) is explicitly declared to extend or implement \(T\), or if \(S\) and \(T\) are the same named type. TypeScript is **primarily structural**, but it exhibits **nominal behavior** in specific situations: class `private` and `protected` members create nominal identity, and `enum` types are nominal in certain respects.

**Beginner-Friendly Explanation.** Nominal typing asks: **“What is this type’s name, and what does it say it inherits from?”** It does not care whether two types happen to have the same shape. If a `Dog` never said it `implements Animal`, then a `Dog` is not an `Animal`, even if it has every method an `Animal` should have.

### Purposes

- **To enforce intentional type design**: a `UserId` and an `OrderId`, even if both are numbers, should not be interchangeable.
- **To provide predictable, declaration-driven subtyping** that does not accidentally accept structurally similar but semantically distinct types.
- **To support encapsulation through private and protected members**, where shape-based compatibility would break invariants.

### Syntax Rules and Structure

TypeScript does not offer a first-class “nominal type” keyword. Nominal behavior emerges from:

1. **`private` and `protected` members** in classes.
2. **`enum` declarations** (nominal in the sense that distinct enums are not interchangeable even if numeric values overlap).
3. **Branded types** (a user-land technique using intersection with a unique marker property).

**General syntax (branded nominal type):**

```
type Brand<T, B> = T & { __brand: B };
type UserId = Brand<number, "UserId">;
```

**Component breakdown:**

- `Brand<T, B>` — a generic type that intersects a base type `T` with a marker property.
- `__brand: B` — a phantom property that exists only at the type level.
- `UserId` — a nominal-like type distinct from plain `number` and from other branded numbers.

**Syntax rules.**

- Two classes with `private` members are compatible only if the private member is declared in the **same** class (or a common superclass).
- Branded types require **type assertions** at creation points, because the phantom property cannot be constructed normally.
- `enum` members are nominally distinct from numeric literals and from other enums.

**Constraints and limitations.**

- Branded types are a **user-land pattern**, not a built-in language feature; they rely on type assertions and must be constructed carefully.
- Nominal class behavior applies only when `private` or `protected` members are present; public-only classes remain structurally compatible.
- TypeScript’s nominal features are **not comprehensive**; there is no `nominal` modifier.

### Annotated Complete Step by Step Code Examples

#### Example 1: Private members creating nominal compatibility

```typescript
// example_nominal.ts
// Step 1: Define two classes with private members.
class Account {
  private id: number;
  constructor(id: number) {
    this.id = id;
  }
}

class Wallet {
  private id: number;
  constructor(id: number) {
    this.id = id;
  }
}

// Step 2: Attempt cross-assignment.
let account: Account = new Account(1);
// let wallet: Wallet = account;
// Error: Property 'id' is private in type 'Account' but not in type 'Wallet'.

console.log("Nominal check passed");
// Expected: Nominal check passed
```

**Expected Output (with invalid assignment commented):**
```
Nominal check passed
```

**Why this output occurs.** Although `Account` and `Wallet` have the same public shape (both have a constructor and a private `id`), they are **not structurally compatible** because their `private` members originate from different declarations. This introduces nominal behavior into TypeScript’s structural system. Without the `private` members, the classes would be compatible.

#### Example 2: Branded types for domain safety

```typescript
// example_branded.ts
// Step 1: Define a brand helper.
type Brand<T, B extends string> = T & { readonly __brand: B };

// Step 2: Define nominal-like IDs.
type UserId = Brand<number, "UserId">;
type OrderId = Brand<number, "OrderId">;

// Step 3: Create branded values via assertion.
const userId = 42 as UserId;
const orderId = 99 as OrderId;

// Step 4: A plain number is assignable to UserId only via assertion.
const anotherUserId: UserId = 7 as UserId;

// Step 5: Cross-assignment is prevented.
// const bad: UserId = orderId;
// Error: Type 'OrderId' is not assignable to type 'UserId'.

console.log(userId);        // Expected: 42
console.log(anotherUserId); // Expected: 7
```

**Expected Output:**
```
42
7
```

**Why this output occurs.** The brand property `__brand` exists only in the type system. At runtime, the values are plain numbers. The compiler treats `UserId` and `OrderId` as distinct because their brand literals differ, preventing accidental interchange of semantically different numeric IDs.

### Real-World Cases

- **Financial systems**: `USD` and `EUR` amounts are branded to prevent mixing currencies.
- **Entity identifiers**: `UserId` and `PostId` are branded to prevent passing the wrong ID to a function.
- **Access control**: `private` class members enforce nominal identity, preventing structurally similar but unauthorized objects from being substituted.

---

## 6. Duck Typing

### Definitions

**Core Definition.** **Duck typing** is a dynamic typing discipline summarized by the principle: “If it walks like a duck and quacks like a duck, then it is a duck.” An object’s suitability for an operation is determined by whether it **responds to the required methods and properties at runtime**, not by its declared type.

**Technical Definition.** Duck typing is a **runtime** form of structural typing. Unlike TypeScript’s structural typing, which is checked at compile time, duck typing checks method and property availability during execution. TypeScript’s documentation describes its structural type checking as “sometimes called ‘duck typing’ or ‘structural subtyping’”, but the two are not identical: structural typing is static; duck typing is dynamic.

**Beginner-Friendly Explanation.** Duck typing means: **“If it can do what I need, I’ll use it — I don’t care what it is called.”** You do not need a formal type declaration. You just try to use the object; if the method exists, it works. If not, you get an error at runtime.

### Purposes

- **To maximize flexibility and code reuse** in dynamically typed code.
- **To enable loose coupling** between components that only need to agree on a small behavioral interface.
- **To support JavaScript patterns** where objects are created dynamically and passed around without formal type declarations.

### Syntax Rules and Structure

Duck typing is **not a syntax**; it is a runtime discipline. In TypeScript, you can **model** duck-typed behavior by:

1. Using `any` or `unknown` and performing runtime checks.
2. Writing user-defined **type guards** that narrow `unknown` to a structural type.
3. Using `typeof` and property checks (`"method" in obj`) at runtime.

**General syntax (type guard):**

```
function isDuck(x: unknown): x is { quack(): void } {
  return typeof x === "object" && x !== null && "quack" in x;
}
```

**Component breakdown:**

- `x: unknown` — accepts any value without static assumptions.
- `x is { quack(): void }` — a **type predicate** that, when `true`, narrows `x` to the structural type.
- `typeof x === "object"` — runtime check that `x` is non-null object.
- `"quack" in x` — runtime check that the `quack` property exists.

**Syntax rules.**

- Type predicates use the `parameterName is Type` syntax in the return type position.
- The predicate function must return a `boolean` (or a type assignable to `boolean`).
- Duck typing itself occurs at runtime; the type guard bridges runtime checks to static narrowing.

**Constraints and limitations.**

- Duck typing provides **no compile-time guarantee** that an object satisfies an interface.
- Runtime checks add overhead and can miss edge cases (e.g., a property exists but is not callable).
- TypeScript’s structural typing is **static** and therefore safer than raw duck typing, but it cannot fully replace runtime duck typing when types are genuinely unknown at compile time.

### Annotated Complete Step by Step Code Examples

#### Example 1: Runtime duck typing with type guards

```typescript
// example_duck.ts
// Step 1: Define a type guard for a duck-like object.
function isQuacker(value: unknown): value is { quack: () => void } {
  return (
    typeof value === "object" &&
    value !== null &&
    "quack" in value &&
    typeof (value as any).quack === "function"
  );
}

// Step 2: Create a duck object.
const duck = {
  quack() {
    console.log("Quack!");
  },
};

// Step 3: Create a non-duck object.
const rock = { hardness: 10 };

// Step 4: Use the type guard to narrow.
if (isQuacker(duck)) {
  duck.quack(); // Expected: Quack!
}

if (!isQuacker(rock)) {
  console.log("Rock does not quack");
  // Expected: Rock does not quack
}
```

**Expected Output:**
```
Quack!
Rock does not quack
```

**Why this output occurs.** At runtime, `isQuacker` checks that `duck` has a callable `quack` property. The type predicate narrows `duck` to `{ quack: () => void }`, allowing the call. `rock` fails the check because it has no `quack` property. This is duck typing mediated by a runtime type guard.

#### Example 2: Contrast with static structural typing

```typescript
// example_duck_contrast.ts
// Step 1: Define a structural interface.
interface Quacker {
  quack(): void;
}

// Step 2: This function uses static structural typing.
function makeItQuack(q: Quacker) {
  q.quack();
}

// Step 3: An object with a quack method is statically accepted.
makeItQuack({
  quack() {
    console.log("Static quack");
  },
});
// Expected: Static quack

// Step 4: An object without quack is rejected at compile time.
// makeItQuack({ noise: "boing" });
// Error: Argument of type '{ noise: string; }' is not assignable to parameter of type 'Quacker'.
```

**Expected Output:**
```
Static quack
```

**Why this output occurs.** The second example uses **static structural typing**: the compiler verifies the shape of the argument before runtime. The first example uses **dynamic duck typing**: the check happens at runtime via the type guard. TypeScript supports both approaches, but its default compatibility rules are static and structural.

### Real-World Cases

- **Polyglot APIs**: A function accepts any object with a `read()` method, regardless of whether it is a file handle, a socket, or a string reader.
- **Plugin systems**: Plugins are duck-typed by the methods they expose (`init`, `execute`, `dispose`), without requiring a shared base class.
- **Testing mocks**: Mock objects in tests satisfy interfaces by shape, even if they are plain objects with jest/vitest mocks, thanks to structural typing.

---

## 7. Type Inference

### Definitions

**Core Definition.** **Type inference** is the process by which the compiler **automatically determines the type** of an expression or variable without an explicit type annotation, using available information such as initializers, return expressions, contextual positions, and best-common-type algorithms.

**Technical Definition.** TypeScript uses a **local, bidirectional** inference algorithm. It infers types from variable initializers, function return expressions, default parameter values, array element candidates (“best common type”), and **contextual typing**, where an expression’s expected type influences the types of its subexpressions. When no best common type exists, the inferred type may be a union.

**Beginner-Friendly Explanation.** Type inference means **the compiler figures out the type for you** when it is obvious from the code. If you write `let x = 3`, TypeScript knows `x` is a `number` without you writing `: number`. You only need to annotate when the type is not obvious or when you want to enforce a specific type.

### Purposes

- **To reduce annotation verbosity** while retaining static safety.
- **To make code easier to read and maintain** by letting types flow from values.
- **To enable contextual typing** in callbacks and higher-order functions, where parameter types are inferred from the expected function type.
- **To support gradual typing**: unannotated code still participates in static checking through inference.

### Syntax Rules and Structure

Type inference has no dedicated syntax; it operates on existing constructs. Key inference sites include:

1. **Variable initialization**: `let x = expr;`
2. **Function return**: `function f() { return expr; }`
3. **Default parameters**: `function f(x = 10) { ... }`
4. **Array literals**: `let arr = [1, 2, 3];`
5. **Contextual positions**: `const handler: (e: Event) => void = (e) => { ... };`

**Component breakdown (contextual inference):**

```
const handler: (event: MouseEvent) => void = (event) => {
  console.log(event.button);
};
```

- `const handler: (event: MouseEvent) => void` — the **contextual type**.
- `(event) => { ... }` — the arrow function; `event` has no annotation.
- The compiler infers `event: MouseEvent` from the contextual type, enabling `event.button`.

**Syntax rules.**

- Inference proceeds from **more specific to less specific** information; explicit annotations override inference.
- When multiple candidate types exist, the **best common type** algorithm selects a type compatible with all candidates.
- If no best common type exists, the result is a **union** of the candidate types.
- Literal types are often **widened** to their base types unless `const` or `as const` preserves literals.

**Constraints and limitations.**

- Inference is **local and syntax-directed**; it does not perform whole-program type inference.
- Over-reliance on inference can make types less obvious to human readers.
- Certain circular or recursive patterns may require explicit annotations.
- Inference may produce `any` in non-strict configurations, defeating safety.

### Annotated Complete Step by Step Code Examples

#### Example 1: Variable and function return inference

```typescript
// example_inference.ts
// Step 1: Inferred from initializer.
let count = 42;               // inferred: number
let message = "hello";        // inferred: string
let isReady = true;           // inferred: boolean

// Step 2: Inferred function return type.
function add(a: number, b: number) {
  return a + b;               // inferred return: number
}

// Step 3: Use the inferred types.
console.log(count);           // Expected: 42
console.log(message);         // Expected: hello
console.log(isReady);         // Expected: true
console.log(add(2, 3));       // Expected: 5
```

**Expected Output:**
```
42
hello
true
5
```

**Why this output occurs.** The compiler infers `number`, `string`, and `boolean` from the initializers. The return type of `add` is inferred as `number` because `a + b` with both `number` operands produces `number`. No explicit annotations are needed for the program to be statically checked.

#### Example 2: Best common type and union inference

```typescript
// example_best_common.ts
// Step 1: Array with mixed numeric and null values.
let values = [0, 1, null];

// Step 2: TypeScript infers (number | null)[] under strictNullChecks.
// The best common type is the union of candidates.

console.log(values.length); // Expected: 3
console.log(values[0]);     // Expected: 0
console.log(values[2]);     // Expected: null
```

**Expected Output:**
```
3
0
null
```

**Why this output occurs.** The candidates are `number` (from `0` and `1`) and `null` (from `null`). No single candidate is a supertype of all others, so the best common type algorithm produces the union `number | null`, and the array type becomes `(number | null)[]`.

#### Example 3: Contextual typing in callbacks

```typescript
// example_contextual.ts
// Step 1: Declare a variable with a function type.
const onClick: (event: { x: number; y: number }) => void = (event) => {
  // Step 2: event is contextually typed; no annotation needed.
  console.log(event.x + event.y);
};

// Step 3: Call with a compatible object.
onClick({ x: 10, y: 20 }); // Expected: 30
```

**Expected Output:**
```
30
```

**Why this output occurs.** The function expression is contextually typed by `onClick`’s declared type. The compiler infers `event` as `{ x: number; y: number }`, so `event.x` and `event.y` are valid. Without contextual typing, `event` would be `any` in non-strict mode and the property accesses would not be statically verified.

### Real-World Cases

- **React event handlers**: Inline handlers inherit parameter types from the JSX attribute’s expected function type.
- **Array transformations**: `[1, 2, 3].map(x => x * 2)` infers `x: number` and the result as `number[]`.
- **Library APIs**: Generic inference lets `identity(42)` infer `T = number` without explicit type arguments.

---

## 8. Type Annotations

### Definitions

**Core Definition.** A **type annotation** is an explicit syntactic construct that associates a type with a variable, parameter, property, or return value. It uses the colon syntax `: Type` in most positions.

**Technical Definition.** Type annotations are **type-level expressions** that the compiler uses during checking and then **erases** during emission. They do not affect runtime behavior. Annotations can appear on variable declarations, function parameters, function return types, class properties, and catch clause variables (with `: unknown` or `: any`).

**Beginner-Friendly Explanation.** A type annotation is **you telling the compiler what type something should be**. Instead of letting TypeScript guess, you write `: string` after a variable name or parameter to enforce that type. Annotations disappear when the code is compiled; they exist only to help the checker and your editor.

### Purposes

- **To override inference** when the inferred type is too wide or not what you intend.
- **To document the public API** of functions, classes, and modules.
- **To enforce constraints at boundaries** where values enter your code from untyped sources.
- **To improve error messages and editor tooling** by providing explicit type information.

### Syntax Rules and Structure

**General syntax (variable annotation):**

```
let identifier: TypeExpression = initializer;
```

**Component breakdown:**

- `let` — declaration keyword.
- `identifier` — name being declared.
- `:` — annotation separator.
- `TypeExpression` — any valid TypeScript type: primitive, union, interface name, generic instantiation, function type, etc.
- `= initializer` — optional; its type must be assignable to the annotation.

**Function parameter and return annotations:**

```
function functionName(param: ParamType): ReturnType {
  // body
}
```

- `param: ParamType` — parameter annotation.
- `): ReturnType` — return type annotation.

**Syntax rules.**

- A space is conventionally placed **after** the colon, not before: `name: string`.
- Return type annotations for arrow functions appear after the parameter list: `(x: number): number => x * 2`.
- Optional properties use `?:` (e.g., `name?: string`).
- Readonly properties use `readonly` before the property name.

**Constraints and limitations.**

- Annotations are **erased** and have no runtime effect.
- An annotation does not perform runtime validation; a value of the wrong type can still enter via `any`, type assertions, or untyped JavaScript.
- Over-annotating can reduce flexibility and duplicate information already available through inference.
- Type assertions (`as T`) are **not** annotations; they override checking rather than declare an expected type.

### Annotated Complete Step by Step Code Examples

#### Example 1: Variable, parameter, and return annotations

```typescript
// example_annotations.ts
// Step 1: Annotate a variable.
let username: string = "Alice";

// Step 2: Annotate function parameters and return type.
function greet(name: string, age: number): string {
  return `Hello, ${name}. You are ${age} years old.`;
}

// Step 3: Annotate an object property inline.
const user: { name: string; active: boolean } = {
  name: username,
  active: true,
};

// Step 4: Call and log.
console.log(greet(username, 30));
// Expected: Hello, Alice. You are 30 years old.
console.log(user.active); // Expected: true
```

**Expected Output:**
```
Hello, Alice. You are 30 years old.
true
```

**Why this output occurs.** The annotations ensure that `username` is a `string`, `greet` receives a `string` and a `number` and returns a `string`, and `user` has exactly the declared properties. The compiler checks these constraints, then erases the annotations. The emitted JavaScript contains only the values and operations.

#### Example 2: Annotation overriding widened inference

```typescript
// example_annotation_override.ts
// Step 1: Without annotation, literal is widened.
let flexible = "hello";          // inferred: string
flexible = "world";              // allowed

// Step 2: With annotation, type is constrained.
let exact: "hello" = "hello";    // type: literal "hello"
// exact = "world";              // Error: Type '"world"' is not assignable to type '"hello"'.

console.log(flexible);           // Expected: world
console.log(exact);              // Expected: hello
```

**Expected Output:**
```
world
hello
```

**Why this output occurs.** `let flexible = "hello"` widens the literal to `string`, allowing reassignment to any string. `let exact: "hello"` preserves the literal type, so reassigning to `"world"` is a compile-time error. Annotations can therefore **narrow** inferred types when needed.

### Real-World Cases

- **Public function signatures**: Library authors annotate exported function parameters and return types to provide stable, documented APIs.
- **Configuration objects**: Interfaces and inline annotations describe expected configuration shapes, enabling editor completion.
- **Boundary validation**: Values from `JSON.parse` are annotated as `unknown` and narrowed with validators, rather than blindly asserted.

---

## 9. Type Checking

### Definitions

**Core Definition.** **Type checking** is the process of verifying that a program respects the rules of its type system — that expressions are used in accordance with their types, that assignments are compatible, and that function calls satisfy parameter and return type requirements.

**Technical Definition.** In TypeScript, type checking is performed by the compiler and language service using the **structural compatibility** relation, along with special rules for functions, generics, unions, intersections, and control-flow-based narrowing. Type checking occurs at **compile time**; there is **no runtime type checking** emitted by default.

**Beginner-Friendly Explanation.** Type checking is the **actual inspection** the compiler performs. Inference and annotations give the compiler information; type checking uses that information to decide whether each operation is legal. When something is wrong, the checker reports an error; when everything is consistent, it emits JavaScript with the types stripped away.

### Purposes

- **To catch illegal operations before runtime**, such as calling a method that does not exist or passing the wrong argument type.
- **To verify that assignments and return values satisfy declared constraints**.
- **To support control-flow-based narrowing** so that `unknown` and union types can be safely used.
- **To power editor diagnostics and code intelligence**.

### Syntax Rules and Structure

Type checking is not a syntax; it is the **semantic process** applied to syntax. The principal checking rules include:

- **Assignability**: `source` is assignable to `target` if every member required by `target` is present in `source` with a compatible type.
- **Function compatibility**: parameters are checked with special rules (fewer parameters are allowed; parameter types are bivariant for methods under default settings, contravariant under `strictFunctionTypes` for function types).
- **Excess property checks**: object literals assigned to a typed target may not contain properties not present in the target type.
- **Control-flow narrowing**: `typeof`, `instanceof`, `in`, equality checks, and user-defined type guards refine types within branches.

**Syntax rules (control-flow narrowing):**

```
function process(value: string | number) {
  if (typeof value === "string") {
    value.toUpperCase();   // narrowed to string
  } else {
    value.toFixed(2);      // narrowed to number
  }
}
```

- `typeof value === "string"` — a **type guard** that narrows `value` in the `if` branch.
- `value.toUpperCase()` — legal only because `value` is narrowed to `string`.
- `value.toFixed(2)` — legal in the `else` branch because `value` is narrowed to `number`.

**Constraints and limitations.**

- TypeScript does **not** perform runtime checking; `any`, type assertions, and untyped JavaScript can bypass static checks.
- The checker is **not fully sound**: bivariant method parameters, `any` propagation, and certain class patterns allow operations that may fail at runtime.
- Control-flow analysis is necessarily conservative; some correct programs may require assertions or restructuring.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic assignability checking

```typescript
// example_checking.ts
// Step 1: Define a target type.
interface Point {
  x: number;
  y: number;
}

// Step 2: Create a source object with extra properties.
const point3D = { x: 1, y: 2, z: 3 };

// Step 3: Assign to Point — allowed structurally.
const point: Point = point3D;

// Step 4: Uncomment to trigger excess property check.
// const direct: Point = { x: 1, y: 2, z: 3 };
// Error: Object literal may only specify known properties.

console.log(point.x + point.y); // Expected: 3
```

**Expected Output:**
```
3
```

**Why this output occurs.** `point3D` has `x` and `y`, so it satisfies `Point` structurally; the extra `z` is ignored because only members required by `Point` are checked. However, an **object literal** assigned directly to `Point` triggers excess property checking, because the literal explicitly includes `z` and `Point` does not declare it.

#### Example 2: Control-flow narrowing with `in`

```typescript
// example_narrowing.ts
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; side: number };

function area(shape: Circle | Square): number {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2; // narrowed to Circle
  } else {
    return shape.side ** 2;             // narrowed to Square
  }
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2));
// Expected: 3.14
console.log(area({ kind: "square", side: 2 }));
// Expected: 4
```

**Expected Output:**
```
3.14
4
```

**Why this output occurs.** The discriminant property `kind` is a literal type in each union member. Checking `shape.kind === "circle"` narrows `shape` to `Circle` inside the first branch; the `else` branch narrows it to `Square`. The compiler can therefore verify that `radius` exists only on `Circle` and `side` only on `Square`.

### Real-World Cases

- **Form validation**: Union types with discriminant properties model form field states (`"valid"`, `"invalid"`, `"pending"`), and type checking ensures each state handles the correct data.
- **API response handling**: Discriminated unions model success and error responses, and narrowing ensures error details are only accessed on error branches.
- **State machines**: Control-flow narrowing tracks the current state and restricts which transitions are statically legal.

---

## 10. Type Erasure

### Definitions

**Core Definition.** **Type erasure** is the process by which the TypeScript compiler **removes all type-level constructs** — annotations, interfaces, type aliases, type parameters, and type-only imports — from the emitted JavaScript, so that the runtime code contains no type information.

**Technical Definition.** Type erasure is a compilation strategy in which types exist only during static checking and are **completely eliminated** in the output. TypeScript’s types are therefore **not reified**; there is no runtime representation of interfaces, type aliases, or generic type arguments. Only runtime constructs such as classes, enums (in some configurations), and values survive compilation.

**Beginner-Friendly Explanation.** Type erasure means **all the type stuff disappears** when your TypeScript becomes JavaScript. Interfaces, annotations, and generic type parameters are like scaffolding around a building: once the building is complete, the scaffolding is taken away. The running JavaScript has no idea what types you wrote — only the values and logic remain.

### Purposes

- **To produce clean, idiomatic JavaScript** that runs on any JavaScript engine without a TypeScript runtime.
- **To impose zero runtime overhead** for type annotations and type-level abstractions.
- **To enable interoperability with the vast JavaScript ecosystem** without requiring type information at runtime.
- **To keep the emitted code close to the source**, simplifying debugging and source maps.

### Syntax Rules and Structure

Type erasure is a **compilation behavior**, not a syntax. The compiler erases:

- Type annotations (`: Type`).
- Interfaces (`interface ... { ... }`).
- Type aliases (`type X = ...`).
- Generic type parameters in declarations and calls (e.g., `function id<T>(x: T): T` becomes `function id(x)`).
- Type-only imports (`import type { Foo } from ...`).
- `as` assertions and non-null assertions (`!`).

The compiler **retains**:

- Classes (as JavaScript classes).
- Enums (unless `const enum` is inlined).
- Namespaces with runtime values.
- Decorators (depending on configuration).

**Component breakdown (erasure of a generic function):**

```
// Source TypeScript
function identity<T>(value: T): T {
  return value;
}
const result = identity<number>(42);
```

```
// Emitted JavaScript (types erased)
function identity(value) {
  return value;
}
const result = identity(42);
```

- `<T>` — type parameter erased.
- `value: T` — annotation erased.
- `: T` — return annotation erased.
- `<number>` — type argument erased.

**Syntax rules.**

- `import type` and `export type` guarantee that an import or export is erased entirely.
- `const enum` members may be inlined at use sites, eliminating the runtime enum object.
- Type assertions (`as T`) are erased completely; they do **not** emit a runtime check.

**Constraints and limitations.**

- **No runtime type checks**: you cannot `instanceof` an interface or query a type alias at runtime.
- **No runtime reflection**: generic type arguments are unavailable at runtime.
- **Type assertions are not casts**: `value as T` does not convert the value; it only tells the compiler to treat it as `T`.
- **Enums may survive erasure** (non-`const` enums emit runtime objects), which is an exception to full erasure.

### Annotated Complete Step by Step Code Examples

#### Example 1: Erasure of annotations and interfaces

```typescript
// example_erasure.ts
// Step 1: Define an interface (type-level only).
interface User {
  name: string;
  age: number;
}

// Step 2: Define a function with annotations.
function formatUser(user: User): string {
  return `${user.name} (${user.age})`;
}

// Step 3: Call the function.
const u: User = { name: "Ada", age: 36 };
console.log(formatUser(u));
// Expected: Ada (36)
```

**Emitted JavaScript (conceptual, types erased):**

```javascript
// interface User { ... }        ← erased entirely
function formatUser(user) {
  return `${user.name} (${user.age})`;
}
const u = { name: "Ada", age: 36 };
console.log(formatUser(u));
```

**Expected Output:**
```
Ada (36)
```

**Why this output occurs.** The `User` interface and all annotations are removed. The runtime code only knows about the object literal and the template string. The behavior is identical because types did not contribute to runtime semantics.

#### Example 2: Type assertions are erased, not checked

```typescript
// example_assertion_erasure.ts
// Step 1: Assert a value to a type.
const value = "hello" as unknown as number;

// Step 2: The assertion is erased; no runtime conversion occurs.
console.log(typeof value); // Expected: string
```

**Expected Output:**
```
string
```

**Why this output occurs.** `as unknown as number` is a **type assertion**, not a runtime cast. The compiler erases the assertion and emits `const value = "hello";`. At runtime, `typeof value` is `"string"`, revealing that the assertion did not change the value’s type.

#### Example 3: Runtime type checking requires a type guard

```typescript
// example_runtime_check.ts
// Step 1: A type guard reconstructs runtime information.
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// Step 2: Use the guard to perform a runtime check.
const data: unknown = "runtime";
if (isString(data)) {
  console.log(data.toUpperCase()); // Expected: RUNTIME
}
```

**Expected Output:**
```
RUNTIME
```

**Why this output occurs.** Because types are erased, the only way to check a type at runtime is to examine the value itself. The `typeof` check inside `isString` provides the runtime evidence, and the type predicate lets the compiler narrow `data` to `string` inside the `if` block.

### Real-World Cases

- **Library distribution**: TypeScript libraries ship `.js` files (types erased) and `.d.ts` declaration files (types preserved for consumers).
- **Node.js execution**: Tools like `ts-node` and `tsx` transpile and erase types on the fly, allowing direct execution of TypeScript with no runtime type layer.
- **Bundle size**: Erasure ensures that interfaces and type aliases add **zero bytes** to production bundles.

---

## 11. Type System Best Practices

### Definitions

**Core Definition.** **Type system best practices** are empirically established guidelines for using TypeScript’s type system in ways that maximize safety, readability, maintainability, and tooling effectiveness while avoiding known pitfalls.

**Technical Definition.** Best practices encompass **strictness configuration**, **type design principles**, **avoidance of unsafe escape hatches**, **preferential use of precise types over wide types**, and **consistent conventions** for annotations, interfaces, generics, and runtime validation. Official guidance is provided in the TypeScript Handbook’s “Do’s and Don’ts” and in community style guides.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to get the most benefit from the type system. They include turning on strict mode, avoiding `any`, using `unknown` when you truly do not know a type, preferring interfaces for public contracts, and validating data at runtime instead of trusting type assertions.

### Purposes

- **To maximize the number of errors caught at compile time** rather than at runtime.
- **To keep types precise and expressive**, avoiding the “type soup” of `any` and overly wide unions.
- **To make code understandable and maintainable** for teams and future maintainers.
- **To ensure safe interoperability** with JavaScript and external data sources.
- **To leverage tooling effectively** by providing accurate type information.

### Syntax Rules and Structure

Best practices are not a single syntax; they are a set of **conventions and compiler configurations**. Key elements include:

- **Enable `--strict`** (or at minimum `--noImplicitAny` and `--strictNullChecks`).
- **Prefer `unknown` over `any`** when a type is genuinely unknown.
- **Use `void` for callbacks whose return value is ignored**.
- **Use lowercase primitive types** (`number`, `string`, `boolean`) rather than boxed types (`Number`, `String`, `Boolean`).
- **Use `import type`** for type-only imports to ensure erasure and avoid runtime dependencies.
- **Brand primitive types** when domain semantics require nominal distinction.
- **Validate external data at runtime** with type guards rather than asserting types.

**Component breakdown (strict configuration in `tsconfig.json`):**

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUncheckedIndexedAccess": true
  }
}
```

- `strict: true` — enables the full family of strict checks.
- `noImplicitAny: true` — errors on implicit `any`.
- `strictNullChecks: true` — distinguishes `null` and `undefined` from other types.
- `strictFunctionTypes: true` — enables contravariant parameter checking for function types.
- `noUncheckedIndexedAccess: true` — adds `undefined` to indexed access results, encouraging guards.

**Syntax rules.**

- Prefer `interface` for object shapes that may be extended or implemented; use `type` for unions, intersections, and aliases of non-object types.
- Use `readonly` for properties that should not be mutated after creation.
- Use discriminated unions with a literal `kind` or `type` property for state modeling.
- Avoid type assertions (`as`) unless you have runtime evidence or are in a migration context.

**Constraints and limitations.**

- Strict mode can surface many errors in existing JavaScript/TypeScript codebases; migration should be incremental.
- Some third-party libraries may lack accurate types, requiring `@types` packages or local declarations.
- `noUncheckedIndexedAccess` can be noisy for code that legitimately relies on array indexing after bounds checks.

### Annotated Complete Step by Step Code Examples

#### Example 1: `unknown` and runtime validation

```typescript
// example_best_unknown.ts
// Step 1: Accept unknown input.
function parseName(input: unknown): string {
  // Step 2: Validate at runtime before using as string.
  if (typeof input !== "string") {
    throw new Error("Expected a string");
  }
  return input.trim();
}

// Step 3: Call with valid and invalid values.
console.log(parseName("  Ada  ")); // Expected: Ada

try {
  parseName(42);
} catch (e) {
  console.log((e as Error).message); // Expected: Expected a string
}
```

**Expected Output:**
```
Ada
Expected a string
```

**Why this output occurs.** `unknown` forces a runtime check before the value can be treated as a `string`. This prevents the unsafe behavior that `any` would allow, where `input.trim()` would compile but fail at runtime. The validation makes the function safe for external input.

#### Example 2: `void` for ignored callback returns

```typescript
// example_best_void.ts
// Step 1: Use void for callbacks whose return is ignored.
function forEachItem<T>(items: T[], callback: (item: T) => void): void {
  for (const item of items) {
    callback(item);
  }
}

// Step 2: Pass a callback that returns a value; it is ignored safely.
forEachItem([1, 2, 3], (n) => n * 2);
// Expected: no output (return values ignored)
```

**Expected Output:**
```
(no output)
```

**Why this output occurs.** `void` communicates that the callback’s return value is intentionally unused. A callback returning `number` is still assignable to `() => void`, but the caller cannot accidentally use the return value. This is safer than `any`, which would allow unchecked use of the return value.

#### Example 3: Discriminated unions for state

```typescript
// example_best_discriminated.ts
// Step 1: Define a discriminated union.
type RequestState =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: string }
  | { status: "error"; message: string };

// Step 2: Exhaustively handle states.
function render(state: RequestState): string {
  switch (state.status) {
    case "idle":
      return "Waiting...";
    case "loading":
      return "Loading...";
    case "success":
      return state.data;        // data is available only here
    case "error":
      return `Error: ${state.message}`;
  }
}

console.log(render({ status: "success", data: "Done" }));
// Expected: Done
console.log(render({ status: "error", message: "Failed" }));
// Expected: Error: Failed
```

**Expected Output:**
```
Done
Error: Failed
```

**Why this output occurs.** The discriminant property `status` allows the compiler to narrow the union in each `switch` branch. `data` is accessible only when `status` is `"success"`, and `message` only when `status` is `"error"`. This pattern eliminates invalid states and makes exhaustive handling checkable.

### Real-World Cases

- **Enterprise codebases**: Strict `tsconfig.json` configurations prevent `any` leakage and null-related bugs.
- **API clients**: Responses are typed as discriminated unions (`{ ok: true; data: T } | { ok: false; error: E }`) for safe error handling.
- **Form libraries**: Field states are modeled as discriminated unions (`idle`, `validating`, `valid`, `invalid`) to ensure correct rendering.
- **Library design**: `import type` and declaration files keep runtime bundles lean while preserving consumer type safety.

---

## References

- TypeScript Handbook — Type Compatibility - https://www.typescriptlang.org/docs/handbook/type-compatibility.html
- TypeScript Handbook — Type Inference - https://www.typescriptlang.org/docs/handbook/type-inference.html
- TypeScript Handbook — Everyday Types (Type Annotations) - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- TypeScript Handbook — Do’s and Don’ts - https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
- TypeScript Handbook — Interfaces (Duck Typing / Structural Subtyping) - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript FAQ — TypeScript Types Are Erased - https://github.com/microsoft/TypeScript/wiki/FAQ
- TypeScript Wiki — Type Erasure (FAQ section) - https://github.com/microsoft/TypeScript/wiki/FAQ#what-is-type-erasure
- TypeScript Documentation — Basic Types (Erased Types) - https://www.typescriptlang.org/docs/handbook/2/basic-types.html
- TypeScript Documentation — TypeScript for JavaScript Programmers - https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html
- TypeScript Documentation — TypeScript for Functional Programmers - https://www.typescriptlang.org/docs/handbook/typescript-for-functional-programmers.html
- TypeScript GitHub Repository — Compiler Source and Design Notes - https://github.com/microsoft/TypeScript
- “Safe & Efficient Gradual Typing for TypeScript” — Rastogi et al., POPL 2015 - https://www.cs.umd.edu/~jfoster/papers/popl15.pdf
- “Gradual Typing” — Siek et al., MIT CSAIL - https://people.csail.mit.edu/rinard/paper/gradual.pdf
- “Typed and Confused: Studying the Unexpected Dangers of Gradual Typing” — IEEE/ACM ASE 2024 - https://ieeexplore.ieee.org/document/10764889
- Google TypeScript Style Guide — Structural Types vs Nominal Types - https://google.github.io/styleguide/tsguide.html
- AWS Prescriptive Guidance — TypeScript Best Practices - https://docs.aws.amazon.com/prescriptive-guidance/latest/best-practices-cdk-typescript-iac/typescript-best-practices.html