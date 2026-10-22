# Type Aliases in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A TypeScript **type alias** is a declaration that assigns a **name to any type expression**, allowing that type to be referenced by the alias throughout the program. Unlike interfaces, which are restricted to object shapes, type aliases can name **primitives, unions, intersections, tuples, function types, mapped types, conditional types, and recursive types**. Type aliases are purely **type-level constructs**—they are erased during compilation and have no runtime representation.

**Technical Definition.** A type alias declaration has the form `type AliasName<TypeParameters> = TypeExpression;`. It introduces a binding into the **type space** (not the value space). The alias is **closed**: unlike interfaces, it cannot be augmented by declaration merging, and its identity is not preserved in error messages—the compiler typically expands the alias to its underlying type during checking and display. Since TypeScript 3.7, type aliases may be **recursive**, allowing them to refer to themselves in their own definition. Type aliases support **generic type parameters** and **constraints** (`extends`), and they are the **only** mechanism for naming union, intersection, and mapped types.

**Beginner-Friendly Explanation.** A type alias is **a nickname for a type**. You can give a short, meaningful name to a long or complex type—like `type UserID = string | number`—and then use `UserID` everywhere instead of repeating the full definition. It works for **any** type: primitives, objects, unions, functions, tuples, and more. Unlike an interface, a type alias is not a “thing” you can add to later; it is a fixed name for a specific type expression. It disappears when the code runs, so it is purely for the type checker and your editor.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Names any type** | Type aliases can name primitives, unions, intersections, tuples, functions, mapped types, conditional types, and recursive types. |
| **Closed (no merging)** | Type aliases **cannot** be augmented via declaration merging; each alias name refers to exactly one type expression. |
| **Erased at runtime** | Type aliases are compile-time only and emit no JavaScript. |
| **Generic support** | Type aliases may declare type parameters with optional constraints: `type Box<T extends string> = { value: T };`. |
| **Recursive (TypeScript 3.7+)** | A type alias may refer to itself in its definition: `type Json = string \| number \| Json[] \| { [key: string]: Json };`. |
| **No identity preservation** | Error messages and IDE displays typically show the **expanded** type, not the alias name, which can reduce readability. |
| **Intersection-based extension** | Type aliases “extend” via intersection types (`type B = A & { extra: string }`), not via an `extends` keyword. |

---

### Prerequisites

- **TypeScript basics**: type annotations, interfaces, and structural typing.
- **Type expressions**: primitives, object types, arrays, tuples, and functions.
- **Generics fundamentals**: type parameters and constraints (`<T extends U>`).
- **The distinction between the type space and the value space** in TypeScript.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compiler behavior.

---

### Related Programming Areas

- **Interface declarations** — the primary alternative for object shapes.
- **Generic programming** — type aliases are heavily used for utility types like `Partial<T>` and `Readonly<T>`.
- **Type-level programming** — conditional types, mapped types, and template literal types rely on type aliases.
- **Union and intersection types** — only expressible through type aliases.
- **Recursive data modeling** — JSON, trees, and linked lists.

---

## Core Concepts / Features

---

## 1. Type Aliases

### Definitions

**Core Definition.** A **type alias** is a named reference to a type expression. It allows a type to be given a descriptive name and reused throughout the program, improving readability and maintainability without introducing runtime code.

**Technical Definition.** A type alias declaration `type Name = TypeExpression` introduces a binding into the type space. The alias may be generic, recursive, and may reference any valid type expression, including unions, intersections, tuples, function types, mapped types, and conditional types. Type aliases are **erased** during compilation. They are **closed**: the same alias name cannot be declared twice in the same scope with different definitions (unlike interfaces, which merge). The TypeScript compiler internally normalizes type aliases, which means error messages often display the expanded type rather than the alias name.

**Beginner-Friendly Explanation.** A type alias is **a name you give to a type**. It is like a variable, but for types instead of values. `type Status = "active" | "inactive"` means you can write `Status` anywhere you need that union, instead of repeating it. It makes code shorter and easier to understand.

### Purposes

- **To reduce repetition** of complex type expressions across a codebase.
- **To give descriptive names to unions, intersections, and tuples** that would otherwise be anonymous.
- **To create reusable utility types** such as `Partial<T>`, `Readonly<T>`, and `Record<K, V>`.
- **To model recursive data structures** like JSON, trees, and linked lists.
- **To serve as the preferred tool for naming any type that is not a plain object shape**.

### Syntax Rules and Structure

**General syntax:**

```
type AliasName<TypeParameters> = TypeExpression;
```

**Component breakdown:**

- `type` — declaration keyword.
- `AliasName` — the alias identifier (PascalCase convention).
- `<TypeParameters>` — optional type parameters with optional constraints.
- `=` — assignment separator.
- `TypeExpression` — any valid TypeScript type expression.
- `;` — terminates the declaration.

**Syntax rules.**

- Type aliases may name **any** type expression: primitives, unions, intersections, tuples, functions, objects, mapped types, and conditional types.
- Type aliases may be **generic** with constraints: `type Box<T extends string> = { value: T };`.
- Type aliases may be **recursive** (TypeScript 3.7+): `type Json = string | number | Json[] | { [key: string]: Json };`.
- Type aliases **cannot** be augmented via declaration merging.
- Type aliases are **hoisted** within their scope and can be referenced before their declaration.

**Constraints and limitations.**

- Type aliases are **closed**; the same name cannot be redeclared in the same scope with a different definition.
- Error messages and IDE displays often show the **expanded** type rather than the alias name, reducing readability.
- Type aliases cannot be used with `instanceof` or any runtime operation.
- Some complex recursive type aliases can cause compiler performance issues.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic type aliases for primitives and unions

```typescript
// example_type_alias_basic.ts
// Step 1: Name a primitive type.
type UserID = string;

// Step 2: Name a union type.
type Status = "active" | "inactive" | "pending";

// Step 3: Use the aliases.
const id: UserID = "user-123";
const status: Status = "active";

console.log(id);     // Expected: user-123
console.log(status); // Expected: active

// const badStatus: Status = "unknown";
// Error: Type '"unknown"' is not assignable to type 'Status'.
```

**Setup.** Compile with `npx tsc example_type_alias_basic.ts --strict`.

**Expected Output:**
```
user-123
active
```

**Why this output occurs.** `UserID` is an alias for `string`; `Status` is an alias for the union `"active" | "inactive" | "pending"`. The compiler checks that `id` is a string and `status` is one of the permitted literals. At runtime, the aliases are erased; only the values remain.

---

#### Example 2: Type alias for an intersection (composition)

```typescript
// example_type_alias_intersection.ts
// Step 1: Define two object types.
type Name = { name: string };
type Age = { age: number };

// Step 2: Compose them via intersection.
type Person = Name & Age;

// Step 3: Use the composed alias.
const person: Person = { name: "Ada", age: 36 };
console.log(person.name); // Expected: Ada
console.log(person.age);  // Expected: 36
```

**Expected Output:**
```
Ada
36
```

**Why this output occurs.** `Person` is an intersection of `Name` and `Age`, requiring both `name` and `age`. The object literal satisfies the combined contract. Type aliases use intersection types where interfaces use `extends`.

---

#### Example 3: Recursive type alias (TypeScript 3.7+)

```typescript
// example_type_alias_recursive.ts
// A recursive type describing JSON values.
type Json =
  | string
  | number
  | boolean
  | null
  | Json[]
  | { [key: string]: Json };

const data: Json = {
  name: "Ada",
  age: 36,
  tags: ["pioneer", "mathematician"],
  address: { city: "London" },
};

console.log(data.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `Json` refers to itself in its own definition, allowing arbitrarily nested JSON structures. This was not possible for type aliases before TypeScript 3.7; it was previously only possible with interfaces. Recursive type aliases are essential for modeling JSON, trees, and linked lists.

---

### Real-World Cases

- **API response unions**: `type ApiResult<T> = { ok: true; data: T } | { ok: false; error: string };`.
- **Domain primitives**: `type UserID = string;`, `type OrderID = string;` for semantic distinction.
- **JSON modeling**: `type Json = string | number | boolean | null | Json[] | { [key: string]: Json };`.
- **Utility types**: `Partial<T>`, `Readonly<T>`, `Record<K, V>`, and `Pick<T, K>` are all implemented as type aliases.

---

## 2. `type` Keyword

### Definitions

**Core Definition.** The **`type` keyword** is the syntactic token that introduces a **type alias declaration**. It signals that the following identifier names a type expression, not a runtime value.

**Technical Definition.** In TypeScript’s grammar, a type alias declaration is `type BindingIdentifier TypeParameters? = Type`. The `type` keyword is a **type-space declaration keyword**; it does not create a value binding. The name introduced is available only in type positions (annotations, generic arguments, `extends`/`implements` clauses). It is distinct from the `interface` keyword, which introduces an open, mergeable object type.

**Beginner-Friendly Explanation.** `type` is **the word you type to start a type alias**. Everything after it—the name, the optional generics, the equals sign, and the type expression—defines the alias. It is the TypeScript equivalent of saying “I am naming a type now.”

### Purposes

- **To introduce a named type alias** into the current scope.
- **To distinguish type aliases from interface declarations** and variable declarations.
- **To signal that the declaration is closed** (non-mergeable) and applies to any type expression.

### Syntax Rules and Structure

**General syntax:**

```
type AliasName = TypeExpression;
```

**Component breakdown:**

- `type` — declaration keyword.
- `AliasName` — binding identifier (PascalCase).
- `=` — assignment separator.
- `TypeExpression` — any valid type expression.

**Syntax rules.**

- The `type` keyword must be followed by a valid identifier.
- The alias name must not conflict with a value in the same scope (though type and value namespaces are separate, confusing overlaps should be avoided).
- The alias is **hoisted** within its scope.
- The alias **cannot** be redeclared with a different definition in the same scope.

**Constraints and limitations.**

- `type` cannot be used to declare a runtime value.
- `type` aliases cannot be merged like interfaces.
- The alias name is erased at runtime; it has no representation in JavaScript.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic `type` keyword usage

```typescript
// example_type_keyword.ts
// Step 1: Use the type keyword.
type Point = { x: number; y: number };

// Step 2: Use the alias as a type.
const p: Point = { x: 1, y: 2 };
console.log(p.x, p.y); // Expected: 1 2
```

**Expected Output:**
```
1 2
```

**Why this output occurs.** The `type` keyword introduces `Point` as an alias for the object type `{ x: number; y: number }`. The object literal is checked against it. At runtime, the alias is erased and only the object remains.

---

#### Example 2: `type` cannot merge

```typescript
// example_type_no_merge.ts
type Box = { width: number };

// ❌ Cannot redeclare a type alias with a different definition.
// type Box = { height: number };
// Error: Duplicate identifier 'Box'.

console.log("Type aliases cannot merge");
```

**Expected Output:**
```
Type aliases cannot merge
```

**Why this output occurs.** Unlike interfaces, type aliases are **closed**. Redeclaring `Box` with a different definition is a compile-time error. This is a fundamental difference between the two constructs.

---

### Real-World Cases

- **Naming complex type expressions** that would be unwieldy inline.
- **Defining utility types** in library code.
- **Creating semantic aliases** for primitives (`type Seconds = number;`).

---

## 3. Type Alias Declaration

### Definitions

**Core Definition.** A **type alias declaration** is the complete syntactic statement that defines a type alias, including the `type` keyword, the alias name, optional type parameters, and the type expression being named.

**Technical Definition.** A type alias declaration introduces a **named type** into the containing declaration space. The declaration may be **generic** (`type Box<T> = { value: T };`), may have **constraints** (`type Box<T extends string> = { value: T };`), and may be **recursive** (`type Json = ... | Json[] | ...;`). The declared type is **structural**: any value with the required shape is compatible. The alias is **erased** during compilation.

**Beginner-Friendly Explanation.** A type alias declaration is **the full statement that defines a type alias**. It includes the `type` keyword, the name, any generic parameters, and the type expression after the equals sign.

### Purposes

- **To define a reusable named type** for any type expression.
- **To parameterize types** with generics for flexibility.
- **To constrain generic parameters** to ensure type safety.
- **To model recursive structures** with self-referential aliases.

### Syntax Rules and Structure

**General syntax (full form):**

```
type AliasName<T extends Constraint = Default> = TypeExpression;
```

**Component breakdown:**

- `type` — keyword.
- `AliasName` — identifier.
- `<T extends Constraint = Default>` — optional type parameters with constraints and defaults.
- `=` — assignment.
- `TypeExpression` — the type being named.

**Syntax rules.**

- Type parameters may have constraints (`extends`) and defaults (`= DefaultType`).
- The type expression may reference the type parameters.
- The type expression may be recursive (refer to the alias itself).
- The declaration is **hoisted** and can be used before its declaration in the same scope.

**Constraints and limitations.**

- The alias cannot be augmented or merged.
- The type expression must be a valid TypeScript type; it cannot reference runtime values except through `typeof`.
- Complex recursive aliases may hit compiler recursion limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic type alias with constraint

```typescript
// example_type_alias_generic.ts
// Step 1: Declare a generic type alias with a constraint.
type StringKeyedBox<T extends string> = {
  key: T;
  value: string;
};

// Step 2: Use with different string literal types.
const nameBox: StringKeyedBox<"name"> = { key: "name", value: "Ada" };
const emailBox: StringKeyedBox<"email"> = { key: "email", value: "ada@example.com" };

console.log(nameBox.key);  // Expected: name
console.log(emailBox.key); // Expected: email

// const badBox: StringKeyedBox<number> = { key: 1, value: "x" };
// Error: Type 'number' does not satisfy the constraint 'string'.
```

**Expected Output:**
```
name
email
```

**Why this output occurs.** `T` is constrained to `string`, so only string literal types (or `string` itself) can be used as type arguments. The alias enforces this constraint at compile time.

---

#### Example 2: Type alias with default type parameter

```typescript
// example_type_alias_default.ts
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

const ok: Result<string> = { success: true, data: "hello" };
const failed: Result<string, { code: number }> = {
  success: false,
  error: { code: 500 },
};

console.log(ok.success);     // Expected: true
console.log(failed.success); // Expected: false
```

**Expected Output:**
```
true
false
```

**Why this output occurs.** The default `E = Error` allows `Result<string>` to be used without specifying an error type. When a custom error type is needed, it can be supplied explicitly. Default type parameters are a feature shared by type aliases and interfaces.

---

### Real-World Cases

- **Generic API wrappers**: `type ApiResponse<T> = { data: T; status: number };`.
- **Utility types**: `type Partial<T> = { [P in keyof T]?: T[P] };`.
- **Recursive models**: `type TreeNode<T> = { value: T; children: TreeNode<T>[] };`.

---

## 4. Type Alias Usage

### Definitions

**Core Definition.** **Type alias usage** refers to the application of a declared type alias in type annotations, generic arguments, `extends`/`implements` clauses, and other type positions throughout a program.

**Technical Definition.** Once declared, a type alias can be used anywhere a type is expected: variable annotations, function parameter and return types, class property types, generic constraints, interface members, and other type aliases. The compiler **expands** the alias to its underlying type during checking, so the alias is transparent to the type system. Aliases can be used with `implements` clauses, but classes cannot be “extended” from a type alias in the same way they extend a class or interface—they can only `implements` object-type aliases.

**Beginner-Friendly Explanation.** Type alias usage is simply **using the alias you defined** wherever you need that type. You write `const user: User = ...` or `function greet(person: Person)`. The alias is a shorthand that makes your code cleaner.

### Purposes

- **To apply named types consistently** across a codebase.
- **To make function signatures and object shapes readable** by using short names instead of long type expressions.
- **To compose types** by using aliases as building blocks in other aliases and interfaces.
- **To parameterize functions, classes, and other types** with generic aliases.

### Syntax Rules and Structure

Type aliases can be used in the following positions:

- **Variable annotations**: `const x: AliasName = value;`
- **Function parameters**: `function f(param: AliasName): void { ... }`
- **Return types**: `function f(): AliasName { ... }`
- **Class properties**: `class C { prop: AliasName; }`
- **Generic arguments**: `const arr: AliasName<number>[] = [];`
- **Interface members**: `interface I { prop: AliasName; }`
- **Type alias definitions**: `type Another = AliasName & { extra: string };`
- **`implements` clauses** (for object-type aliases): `class C implements AliasName { ... }`

**Syntax rules.**

- The alias must be in scope where it is used.
- Generic aliases must be instantiated with the correct number of type arguments (unless defaults are provided).
- Object-type aliases can be used in `implements` clauses; non-object aliases (unions, primitives) cannot.
- Type aliases cannot be used as base classes in `extends` clauses; use `implements` for object aliases or intersection for composition.

**Constraints and limitations.**

- Type aliases cannot be extended with `extends`; they are composed with intersection (`&`).
- Union aliases cannot be used in `implements` clauses.
- Aliases are erased; they provide no runtime reflection.

### Annotated Complete Step by Step Code Examples

#### Example 1: Alias in function signatures

```typescript
// example_alias_usage_function.ts
type UserID = string;
type User = { id: UserID; name: string };

function findUser(id: UserID): User | undefined {
  return id === "u1" ? { id, name: "Ada" } : undefined;
}

const user = findUser("u1");
console.log(user?.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `UserID` and `User` are used as parameter and return types. The compiler expands the aliases during checking. The function signature is concise and readable.

---

#### Example 2: Alias in generic arguments

```typescript
// example_alias_usage_generic.ts
type UserID = string;

const ids: UserID[] = ["u1", "u2", "u3"];
const idSet: Set<UserID> = new Set(ids);

console.log(idSet.size); // Expected: 3
```

**Expected Output:**
```
3
```

**Why this output occurs.** `UserID` is used as a type argument to `Array` and `Set`. The aliases are expanded to `string`, and the compiler checks that all elements are strings.

---

#### Example 3: Alias in `implements` clause (object-type alias)

```typescript
// example_alias_usage_implements.ts
type Named = { name: string };

class Person implements Named {
  name: string = "Ada";
}

const person = new Person();
console.log(person.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The object-type alias `Named` is used in the `implements` clause. The compiler checks that `Person` has a `name: string` property. Union aliases cannot be used in `implements` clauses.

---

### Real-World Cases

- **API client code**: `type ApiResult<T> = ...` used throughout service layers.
- **Component props**: `type ButtonProps = { label: string; onClick: () => void };` used in React components.
- **Domain models**: `type UserID = string;` used in function signatures and object shapes.

---

## 5. Type Alias vs Interface

### Definitions

**Core Definition.** **Type aliases** and **interfaces** are the two primary constructs in TypeScript for naming types. They are **very similar** for common object-shape use cases and are often interchangeable, but they differ in **extensibility, mergeability, the range of types they can represent, and the quality of error messages**.

**Technical Definition.** Both type aliases and interfaces can describe object shapes, be generic, and be used in type annotations. Key differences: interfaces are **open** (mergeable via declaration merging) and support `extends`; type aliases are **closed** and compose via intersection (`&`). Interfaces can only describe **object types** (and function/constructor types with call/construct signatures), while type aliases can name **any** type, including unions, intersections, primitives, tuples, and mapped/conditional types. Interfaces generally produce **better error messages** and **better compiler performance** for object shapes; type aliases are mandatory for non-object types.

**Beginner-Friendly Explanation.** Type aliases and interfaces are **two ways to name a type**. For simple objects, they often look identical. The main differences are: interfaces can be **added to** later (declaration merging), while type aliases cannot; interfaces use `extends` to build on other interfaces, while type aliases use `&`; and type aliases can name **any** type (including unions and primitives), while interfaces can only describe objects. For plain object shapes, the TypeScript team recommends interfaces because they produce clearer error messages.

### Purposes

- **To choose the right construct** for each type-naming scenario.
- **To leverage interface merging** when augmenting third-party types.
- **To use type aliases for unions, intersections, and utility types** that interfaces cannot express.
- **To maximize error message clarity and tooling performance** for object shapes.

### Syntax Rules and Structure

**Comparison table:**

| Feature | `interface` | `type` |
|---|---|---|
| **Object shapes** | ✅ Yes | ✅ Yes |
| **Unions** | ❌ No | ✅ Yes |
| **Intersections** | ❌ No (uses `extends`) | ✅ Yes (`&`) |
| **Primitives** | ❌ No | ✅ Yes |
| **Tuples** | ❌ No | ✅ Yes |
| **Mapped types** | ❌ No | ✅ Yes |
| **Conditional types** | ❌ No | ✅ Yes |
| **Declaration merging** | ✅ Yes | ❌ No |
| **Extending** | `extends` | Intersection (`&`) |
| **Error message quality** | Better for objects | Expanded, sometimes worse |
| **Compiler performance** | Generally better for objects | Can be slower for complex types |

**Syntax rules.**

- Use `interface` for object shapes, especially public APIs.
- Use `type` for unions, intersections, primitives, tuples, mapped types, conditional types, and recursive types.
- Both can be used in `implements` clauses (for object types).
- Type aliases cannot be augmented via declaration merging; interfaces can.

**Constraints and limitations.**

- Interfaces cannot express unions, intersections, or mapped types directly.
- Type aliases can produce harder-to-read error messages because they are expanded.
- Declaration merging (interfaces) can be surprising; use `type` when merging is undesirable.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface vs type for object shapes

```typescript
// example_interface_vs_type.ts
// ✅ Interface for an object shape.
interface UserInterface {
  name: string;
  age: number;
}

// ✅ Type alias for the same shape (legal, but interface is preferred).
type UserType = {
  name: string;
  age: number;
};

const u1: UserInterface = { name: "Ada", age: 36 };
const u2: UserType = { name: "Grace", age: 85 };

console.log(u1.name); // Expected: Ada
console.log(u2.name); // Expected: Grace
```

**Expected Output:**
```
Ada
Grace
```

**Why this output occurs.** Both constructs describe the same object shape and are structurally compatible. The TypeScript Handbook and Google Style Guide recommend **interfaces** for object shapes because they produce clearer error messages and better tooling performance.

---

#### Example 2: Type alias for a union (interfaces cannot do this)

```typescript
// example_type_union.ts
// ✅ Type alias for a union.
type Status = "active" | "inactive" | "pending";

const status: Status = "active";
console.log(status); // Expected: active

// ❌ An interface cannot express this union.
// interface Status { ... } // Not possible for a union.
```

**Expected Output:**
```
active
```

**Why this output occurs.** Unions are **only** expressible via type aliases. Interfaces are restricted to object shapes. This is a fundamental reason to use `type` for unions and other non-object types.

---

#### Example 3: Declaration merging (interfaces only)

```typescript
// example_declaration_merging.ts
interface Box {
  width: number;
}

// Second declaration merges with the first.
interface Box {
  height: number;
}

const box: Box = { width: 10, height: 20 };
console.log(box.width + box.height); // Expected: 30

// ❌ Type aliases cannot merge.
// type Box = { width: number };
// type Box = { height: number }; // Error: Duplicate identifier.
```

**Expected Output:**
```
30
```

**Why this output occurs.** Interfaces support **declaration merging**: two `interface Box` declarations merge into one type with both properties. Type aliases are **closed** and cannot be augmented. This is the key reason to choose interfaces when third-party type augmentation is needed.

---

### Real-World Cases

- **Public object APIs**: interfaces are preferred for clearer error messages and mergeability.
- **Union-based domain modeling**: type aliases are required for discriminated unions and literal unions.
- **Third-party type augmentation**: interfaces allow adding properties to `Window`, `ProcessEnv`, and library types.
- **Utility type definitions**: type aliases implement `Partial<T>`, `Readonly<T>`, and other mapped types.

---

## 6. Type Alias Best Practices

### Definitions

**Core Definition.** **Type alias best practices** are guidelines for using type aliases appropriately—choosing them over interfaces when they are the better tool, avoiding their pitfalls, and keeping type definitions clear, safe, and maintainable.

**Technical Definition.** Best practices center on the principle articulated by the TypeScript team and Google Style Guide: **“Use interfaces for object shapes; use type aliases for everything else.”** Specifically: prefer `interface` for object literals and public APIs (better error messages, declaration merging, performance); use `type` for unions, intersections, primitives, tuples, mapped types, conditional types, and recursive types. Do not prefix type aliases with `I` or `T`; use PascalCase. Avoid putting `| null` or `| undefined` in type aliases (add nullability at usage sites). Keep aliases focused and well-named, and document public aliases.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep type aliases clean and safe. The big rules are: use `interface` for object shapes and `type` for everything else; do not create a type alias for an object when an interface would do; do not add `null` or `undefined` to aliases—handle those at the usage site; and give aliases clear, descriptive names.

### Purposes

- **To choose the right construct** between `interface` and `type`.
- **To improve error messages and tooling performance** for object shapes.
- **To avoid anti-patterns** such as I-prefixed aliases and null-inclusive unions.
- **To keep type definitions focused, readable, and maintainable**.
- **To leverage type aliases for their unique strengths**: unions, intersections, and utility types.

### Syntax Rules and Structure

Best practices are conventions that interact with type alias syntax. The core rules are:

1. **Prefer `interface` for object shapes.** Use `type` only when you need unions, intersections, tuples, primitives, mapped types, or conditional types. Google’s TypeScript Style Guide states: “When declaring types for objects, use interfaces instead of a type alias for the object literal expression.”
2. **Use `type` for unions, intersections, and utility types.** These are impossible or unwieldy with interfaces.
3. **Do not prefix type aliases with `I` or `T`.** Use PascalCase: `type User = ...`, not `type IUser = ...` or `type TUser = ...`.
4. **Do not include `| null` or `| undefined` in type aliases.** Add nullability at the usage site. Prefer optional properties over `| undefined`.
5. **Use `type` for recursive types and mapped types** where interfaces are not suitable.
6. **Keep aliases focused and descriptive.** A good alias name explains the type’s purpose.
7. **Document public aliases.** For library APIs, explain the intent of the alias in comments or documentation.
8. **Avoid overly complex type aliases** that are hard to understand; prefer composition of simpler aliases.

**Component breakdown (interface vs type decision tree):**

```
Is the type an object shape used as a public contract?
  ├── Yes → Prefer interface.
  └── No → Is it a union, intersection, primitive, tuple, mapped type, or conditional type?
           ├── Yes → Use type.
           └── No → Use type.
```

**Constraints and limitations.**

- Interfaces cannot express unions, intersections, or mapped types directly.
- Type aliases are expanded in error messages, which can reduce clarity.
- Declaration merging (interfaces) can be surprising; use `type` when merging is undesirable.
- Null-inclusive aliases can hide nullability bugs; handle at the usage site.

### Annotated Complete Step by Step Code Examples

#### Example 1: Prefer interface for object shapes, type for unions

```typescript
// example_best_choose.ts
// ✅ Interface for an object shape.
interface User {
  id: string;
  name: string;
}

// ✅ Type for a union.
type UserRole = "admin" | "editor" | "viewer";

// ✅ Type for an intersection (composition).
type AdminUser = User & { role: "admin" };

// ✅ Type for a mapped/utility type.
type PartialUser = Partial<User>;

const user: User = { id: "u1", name: "Ada" };
const role: UserRole = "admin";

console.log(user.name); // Expected: Ada
console.log(role);      // Expected: admin
```

**Expected Output:**
```
Ada
admin
```

**Why this output occurs.** The object shape uses an interface; the union, intersection, and mapped type use `type`. This follows the TypeScript Handbook’s recommendation to use `interface` for objects and `type` for other type expressions.

---

#### Example 2: Avoiding I-prefix and null-inclusive aliases

```typescript
// example_best_naming_null.ts
// ❌ Avoid: I-prefixed alias.
// type IUser = { name: string };

// ✅ Correct: no prefix.
type User = { name: string };

// ❌ Avoid: null-inclusive alias.
// type MaybeUser = User | null;

// ✅ Correct: handle nullability at the usage site.
function getUser(): User | null {
  return { name: "Ada" };
}

const user = getUser();
if (user) {
  console.log(user.name); // Expected: Ada
}
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The alias is named `User`, not `IUser`, following community conventions. Nullability is handled in the function’s return type, not baked into the alias. This keeps the alias focused and reusable.

---

#### Example 3: Using type for recursive and mapped types

```typescript
// example_best_recursive_mapped.ts
// ✅ Recursive type alias for JSON.
type Json = string | number | boolean | null | Json[] | { [key: string]: Json };

// ✅ Mapped type alias.
type ReadonlyDeep<T> = {
  readonly [K in keyof T]: T[K] extends object ? ReadonlyDeep<T[K]> : T[K];
};

type User = { name: string; address: { city: string } };
const user: ReadonlyDeep<User> = {
  name: "Ada",
  address: { city: "London" },
};

console.log(user.address.city); // Expected: London
```

**Expected Output:**
```
London
```

**Why this output occurs.** Recursive types and mapped types are expressible **only** through type aliases, not interfaces. These are the primary use cases for `type` over `interface`.

---

### Real-World Cases

- **Library API design**: interfaces for public object contracts; type aliases for unions and utility types.
- **Domain modeling**: `type OrderState = "pending" | "shipped" | "delivered";` (union), `interface Order { ... }` (object).
- **Utility type libraries**: `Partial<T>`, `Readonly<T>`, and `Record<K, V>` are type aliases.
- **JSON and recursive data**: `type Json = ...` is a type alias.

---

## References

- TypeScript Handbook — Type Aliases (Everyday Types) - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases
- TypeScript Handbook — Object Types (Interfaces vs. Type Aliases) - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript 4.2 Release Notes — Smarter Type Alias Preservation - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-2.html
- TypeScript 3.7 Release Notes — Recursive Type Aliases - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html
- TypeScript Playground — Types vs Interfaces - https://www.typescriptlang.org/play/typescript/meta-types/types-vs-interfaces.ts.html
- TypeScript Playground — Recursive Type References - https://www.typescriptlang.org/play/3-7/types-and-code-flow/recursive-type-references.ts.html
- TypeScript Documentation — Declaration Reference (Reusable Types: Type Aliases) - https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html#reusable-types-type-aliases
- Google TypeScript Style Guide — Prefer Interfaces Over Type Literal Aliases - https://google.github.io/styleguide/tsguide.html
- Google TypeScript Style Guide — Aliases (Nullability) - https://google.github.io/styleguide/tsguide.html
- TypeScript Handbook — Generics (Generic Type Aliases) - https://www.typescriptlang.org/docs/handbook/2/generics.html
- W3Schools — TypeScript Type Aliases and Interfaces - https://www.w3schools.com/typescript/typescript_aliases_and_interfaces.php
- LogRocket Blog — Understanding and Using Interfaces in TypeScript (Type Aliases vs Interfaces) - https://blog.logrocket.com/understanding-using-interfaces-in-typescript/
- Total TypeScript — Type Aliases vs Interfaces - https://www.totaltypescript.com/type-aliases-vs-interfaces
- pproenca/dot-skills — Prefer Interfaces Over Type Aliases for Objects - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/ts-google/references/types-prefer-interfaces.md