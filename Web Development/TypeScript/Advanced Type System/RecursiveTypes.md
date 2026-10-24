# Recursive Types in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **recursive type** is a type definition that **refers to itself**, directly or indirectly, within its own structure. Recursive types model **self-similar data**—trees, linked lists, JSON, nested configurations, and other structures of unbounded depth. TypeScript supports recursion through **type aliases**, **interfaces**, **recursive conditional types**, and **homomorphic mapped types**.

**Technical Definition.** Before TypeScript 3.7, type aliases could not refer to themselves at the top level; only interfaces could be recursive. TypeScript 3.7 lifted this restriction by **deferring resolution of type arguments** at the top level of a type alias, permitting recursive aliases such as `type Json = string | number | boolean | null | Json[] | { [key: string]: Json }`. TypeScript 4.1 further enabled **recursive conditional types**, allowing conditional types to reference themselves within their branches, which powers deep transformations like `DeepPartial`, `DeepReadonly`, and `Awaited`. Recursive types are **compile-time only**; they emit no runtime code, but deep or unbounded recursion can cause **exponential type expansion**, **compiler hangs**, and **“Type instantiation is excessively deep”** errors.

**Beginner-Friendly Explanation.** A recursive type is **a type that contains itself**. A tree is the classic example: a tree node has a value and a list of child trees—each child is also a tree. TypeScript lets you describe this self-similar shape with a type that references its own name. Before TypeScript 3.7, this was only possible with interfaces; now type aliases can be recursive too. The danger is that **unbounded recursion can overwhelm the compiler**, so recursive types need clear **termination conditions** and often **depth limits**.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Self-reference** | The type definition mentions itself directly or through another type. |
| **Type aliases (TS 3.7+)** | `type` aliases may be recursive when resolution can be deferred (objects, arrays, tuples, generics). |
| **Interfaces (always)** | Interfaces have always supported recursive references and remain the traditional choice for object trees. |
| **Recursive conditional types (TS 4.1+)** | Conditional types can name themselves in their branches, enabling deep structural transformations. |
| **Homomorphic recursion** | Mapped types over `keyof T` recurse naturally while preserving modifiers and array/tuple structure. |
| **Termination** | Recursion must terminate on primitives, functions, and other base cases. |
| **Depth limits** | TypeScript’s instantiation depth limit is roughly **50 levels**; deeper recursion causes errors and IDE lag. |
| **Compile-time only** | Recursive types are erased and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type aliases, interfaces, unions, and generics.
- **Conditional types** and the **`infer` keyword**.
- **Mapped types** and **`keyof`**.
- **Indexed access types** (`T[K]`).
- **The `extends` keyword** in constraints and conditional types.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Tree and graph modeling** — recursive structures for hierarchical data.
- **JSON and API modeling** — recursive JSON types and API response shapes.
- **Deep utility types** — `DeepPartial`, `DeepReadonly`, `DeepRequired`, `Awaited`.
- **Type-level programming** — recursive conditional and mapped types.
- **Compiler performance** — recursion depth and exponential expansion are the primary constraints.

---

## Core Concepts / Features

---

## 1. Recursive Types

### Definitions

**Core Definition.** A **recursive type** is a type whose definition includes a reference to itself, allowing it to describe **self-similar structures of arbitrary depth**.

**Technical Definition.** Recursion is achieved through **type aliases** (`type Tree = { children: Tree[] }`), **interfaces** (`interface Tree { children: Tree[] }`), **recursive conditional types** (`T extends object ? DeepPartial<T[K]> : T`), and **homomorphic mapped types** (`{ [K in keyof T]: DeepPartial<T[K]> }`). TypeScript 3.7 relaxed the rules for type alias recursion by deferring resolution of type arguments at the top level, and TypeScript 4.1 allowed conditional types to reference themselves within their branches.

**Beginner-Friendly Explanation.** A recursive type is **a type that is defined in terms of itself**. It is how you describe things that can nest inside themselves, like folders inside folders or comments with replies.

### Purposes

- **To model self-similar data** such as trees, linked lists, JSON, and nested configurations.
- **To describe hierarchical structures** of unbounded depth without manually enumerating levels.
- **To enable deep type transformations** that apply recursively to every nested level.
- **To keep type definitions synchronized** with recursive runtime data shapes.
- **To replace `any` or `unknown`** in recursive data models with precise, checked types.

### Syntax Rules and Structure

**Recursive type alias (TS 3.7+):**

```typescript
type Json = string | number | boolean | null | Json[] | { [key: string]: Json };
```

**Recursive interface:**

```typescript
interface TreeNode {
  value: string;
  children: TreeNode[];
}
```

**Recursive conditional type (TS 4.1+):**

```typescript
type DeepReadonly<T> = T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;
```

**Component breakdown:**

- `Json` — the recursive alias; each member refers back to `Json`.
- `children: TreeNode[]` — an array of the same interface type.
- `DeepReadonly<T[K]>` — recursive application inside the mapped type.

**Syntax rules.**

- Type aliases may be recursive when the reference is deferred through an object, array, tuple, or generic position.
- Interfaces support recursion natively through property types.
- Recursive conditional types must have a **base case** that terminates (primitives, functions, `never`).
- Homomorphic mapped types (`{ [K in keyof T]: ... }`) recurse naturally and preserve array/tuple structure.

**Constraints and limitations.**

- Deep recursion beyond **~50 levels** causes exponential compiler work and “Type instantiation is excessively deep” errors.
- Cyclic types (types that refer to themselves through a cycle without a base case) do not terminate.
- Unbounded recursive mapped types like `DeepPartial` without depth limits can hang the compiler on deeply nested objects.

### Annotated Complete Step by Step Code Examples

#### Example 1: Recursive JSON type

```typescript
// example_recursive_json.ts
// Step 1: Define a recursive JSON type.
type Json =
  | string
  | number
  | boolean
  | null
  | Json[]
  | { [key: string]: Json };

// Step 2: Use it with nested data.
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

**Why this output occurs.** The `Json` alias refers to itself in the array and object members, allowing arbitrary nesting. The object literal is checked recursively against `Json`.

---

#### Example 2: Recursive interface tree

```typescript
// example_recursive_tree.ts
// Step 1: Define a recursive tree interface.
interface TreeNode {
  value: string;
  children: TreeNode[];
}

// Step 2: Create a nested tree.
const tree: TreeNode = {
  value: "root",
  children: [
    { value: "child1", children: [] },
    { value: "child2", children: [{ value: "grandchild", children: [] }] },
  ],
};

console.log(tree.children[1].children[0].value); // Expected: grandchild
```

**Expected Output:**
```
grandchild
```

**Why this output occurs.** `TreeNode` references itself through `children`, allowing the structure to nest arbitrarily. The compiler checks every level against the interface.

---

### Real-World Cases

- **JSON modeling**: `type Json = ...` for API payloads and configuration files.
- **File systems**: folders containing files and subfolders.
- **Comment threads**: comments with nested replies.
- **Organization charts**: employees with direct reports.

---

## 2. Recursive Type Declaration

### Definitions

**Core Definition.** A **recursive type declaration** is the statement that defines a recursive type—using `type` or `interface`—with a self-reference in its structure.

**Technical Definition.** Recursive declarations use **type aliases** (supported since TypeScript 3.7 for deferred positions) or **interfaces** (always supported). The declaration must allow the compiler to **defer resolution** of the recursive reference; this is why type aliases could not be recursive at the top level before 3.7. Recursive conditional types (TypeScript 4.1+) may also be declared as recursive aliases whose branches reference the alias.

**Beginner-Friendly Explanation.** A recursive type declaration is **the line of code where you define a self-referential type**. You choose `type` or `interface` and write a structure that includes itself.

### Purposes

- **To define the shape of recursive data** once, without enumerating every depth.
- **To choose between `type` and `interface`** based on extension and merging needs.
- **To enable recursive conditional transformations** through self-referencing aliases.
- **To document the intended nesting structure** clearly.

### Syntax Rules and Structure

**Type alias declaration:**

```typescript
type RecursiveType = BaseCase | { child: RecursiveType };
```

**Interface declaration:**

```typescript
interface RecursiveInterface {
  child: RecursiveInterface;
}
```

**Recursive conditional declaration:**

```typescript
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;
```

**Syntax rules.**

- Type aliases may be recursive when the reference is inside an object, array, tuple, or generic position.
- Interfaces are recursive by default through property types.
- Recursive conditional type aliases require a **terminating base case**.
- Prefer `interface` when you need declaration merging or extension; prefer `type` for unions and computed recursive shapes.

**Constraints and limitations.**

- Top-level direct self-reference in a type alias (e.g., `type T = T`) is still invalid.
- Recursive aliases with conditional types can hit instantiation limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: Type alias vs. interface recursion

```typescript
// example_declaration_forms.ts
// ✅ Type alias recursion (TS 3.7+).
type TreeAlias = { value: string; children: TreeAlias[] };

// ✅ Interface recursion (always supported).
interface TreeInterface {
  value: string;
  children: TreeInterface[];
}

const tree: TreeAlias = { value: "root", children: [] };
console.log(tree.value); // Expected: root
```

**Expected Output:**
```
root
```

**Why this output occurs.** Both forms allow the type to reference itself. The type alias relies on the TS 3.7 relaxation; the interface has always supported recursion.

---

### Real-World Cases

- **Choosing `interface`** for extensible trees and declaration merging.
- **Choosing `type`** for recursive unions like JSON and computed recursive aliases.

---

## 3. Recursive Type Usage

### Definitions

**Core Definition.** **Recursive type usage** is the application of recursive types in annotations, function signatures, generic constraints, and transformations that operate on nested data.

**Technical Definition.** Recursive types are used wherever nested data must be typed: **function parameters and return types**, **generic constraints**, **mapped types**, and **recursive conditional types**. Recursive conditional types **walk into nested structures** and apply a transformation at every level, powering utilities like `DeepReadonly` and `DeepPartial`.

**Beginner-Friendly Explanation.** You use recursive types **anywhere data can nest**—when writing functions that traverse trees, define deeply nested configuration objects, or transform every level of an object.

### Purposes

- **To type recursive data structures** in functions and APIs.
- **To apply transformations at every nesting level** with recursive conditional types.
- **To constrain generic parameters** to recursive shapes.
- **To traverse and validate nested data** with type safety.

### Syntax Rules and Structure

**Recursive function parameter:**

```typescript
function walk(node: TreeNode): void {
  console.log(node.value);
  node.children.forEach(walk);
}
```

**Recursive generic constraint:**

```typescript
type RecursiveRecord<T> = { [K in keyof T]: T[K] extends object ? RecursiveRecord<T[K]> : T[K] };
```

**Recursive conditional transformation:**

```typescript
type DeepReadonly<T> = T extends (infer U)[]
  ? readonly DeepReadonly<U>[]
  : T extends Function
    ? T
    : T extends object
      ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
      : T;
```

**Syntax rules.**

- Recursive functions follow the same structural typing as the recursive type.
- Recursive conditional types require base cases for arrays, functions, objects, and primitives.
- The order of conditional branches matters: check arrays before objects, functions before objects, and terminate on primitives.

**Constraints and limitations.**

- Recursive traversal at runtime is fine; recursive **type computation** is what hits compiler limits.
- Functions should terminate on primitives to avoid infinite recursion.

### Annotated Complete Step by Step Code Examples

#### Example 1: Recursive tree traversal

```typescript
// example_usage_traverse.ts
interface TreeNode {
  value: string;
  children: TreeNode[];
}

function countNodes(node: TreeNode): number {
  return 1 + node.children.reduce((sum, child) => sum + countNodes(child), 0);
}

const tree: TreeNode = {
  value: "root",
  children: [
    { value: "a", children: [] },
    { value: "b", children: [{ value: "c", children: [] }] },
  ],
};

console.log(countNodes(tree)); // Expected: 4
```

**Expected Output:**
```
4
```

**Why this output occurs.** The function recurses into `children`, counting every node. The recursive type ensures every child is a valid `TreeNode`.

---

#### Example 2: Recursive transformation with `DeepReadonly`

```typescript
// example_usage_deep_readonly.ts
type DeepReadonly<T> = T extends (infer U)[]
  ? readonly DeepReadonly<U>[]
  : T extends Function
    ? T
    : T extends object
      ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
      : T;

interface Config {
  database: { host: string; port: number };
  features: string[];
}

const config: DeepReadonly<Config> = {
  database: { host: "localhost", port: 5432 },
  features: ["a", "b"],
};

// config.database.host = "other"; // ❌ Error: readonly.
console.log(config.database.host); // Expected: localhost
```

**Expected Output:**
```
localhost
```

**Why this output occurs.** The recursive conditional type descends into objects and arrays, marking every level `readonly`. Mutating nested properties is rejected at compile time.

---

### Real-World Cases

- **Tree traversal utilities**: counting, mapping, and filtering tree nodes.
- **Deep freeze utilities**: applying `DeepReadonly` to configuration objects.
- **Deep partial updates**: applying `DeepPartial` to nested update payloads.

---

## 4. Recursive Aliases

### Definitions

**Core Definition.** A **recursive alias** is a type alias that refers to itself in its definition. Since TypeScript 3.7, recursive aliases are permitted when the self-reference is **deferred** through an object, array, tuple, or generic position.

**Technical Definition.** TypeScript 3.7 relaxed the restriction on recursive type aliases by deferring resolution of type arguments at the top level. This enables comprehensive recursive definitions such as `type Json = string | number | boolean | null | Json[] | { [key: string]: Json }`. Before 3.7, the workaround was to mix a type alias with an interface that extends an array type.

**Beginner-Friendly Explanation.** A recursive alias is **a `type` that mentions itself**. It is the type-alias version of a recursive interface. Since TypeScript 3.7, you can write recursive aliases directly without the old interface workaround.

### Purposes

- **To define recursive unions and JSON-like structures** with `type`.
- **To avoid the interface workaround** required before TypeScript 3.7.
- **To combine recursion with unions and conditional types** that interfaces cannot express.
- **To build recursive utility types** like `DeepPartial` and `Paths<T>`.

### Syntax Rules and Structure

**Basic recursive alias:**

```typescript
type Json = string | number | boolean | null | Json[] | { [key: string]: Json };
```

**Recursive alias with generics:**

```typescript
type Tree<T> = { value: T; children: Tree<T>[] };
```

**Syntax rules.**

- The self-reference must be in a **deferred position** (object, array, tuple, or generic).
- Direct top-level self-reference (`type T = T`) remains invalid.
- Recursive aliases may be combined with unions and conditional types.

**Constraints and limitations.**

- Recursive aliases can produce complex error messages.
- Deep recursion can hit compiler limits.

### Annotated Complete Step by Step Code Examples

#### Example 1: Recursive JSON alias

```typescript
// example_alias_json.ts
type Json = string | number | boolean | null | Json[] | { [key: string]: Json };

const data: Json = { a: [1, true, { b: null }] };
console.log(typeof data); // Expected: object
```

**Expected Output:**
```
object
```

**Why this output occurs.** The alias refers to itself in the array and object members, allowing arbitrary nesting. The entire structure is checked recursively.

---

#### Example 2: Generic recursive alias

```typescript
// example_alias_generic.ts
type Tree<T> = { value: T; children: Tree<T>[] };

const tree: Tree<number> = {
  value: 1,
  children: [{ value: 2, children: [] }],
};

console.log(tree.children[0].value); // Expected: 2
```

**Expected Output:**
```
2
```

**Why this output occurs.** The generic alias `Tree<T>` refers to itself with the same type argument, preserving the element type at every level.

---

### Real-World Cases

- **JSON and API payloads**: recursive JSON aliases.
- **Generic trees**: `Tree<T>` for ASTs, file systems, and organization charts.
- **Recursive unions**: expression trees and algebraic data types.

---

## 5. Tree Structures

### Definitions

**Core Definition.** A **tree structure type** is a recursive type in which a node contains a **collection of child nodes of the same type**, modeling hierarchical data.

**Technical Definition.** Tree types are typically defined with an interface or type alias where a node has a `children` property of type `NodeType[]`. They may be **generic** (`Tree<T>`) to parameterize the value type. Tree types support recursive traversal, mapping, and folding operations at runtime.

**Beginner-Friendly Explanation.** A tree type is **a node that has children, and each child is also a node**. It is the natural way to model folders, comments, ASTs, and organization charts.

### Purposes

- **To model hierarchical data** such as file systems, comment threads, and ASTs.
- **To support recursive traversal** with type-safe child access.
- **To enable generic tree transformations** with `Tree<T>`.
- **To validate nested structures** at every level.

### Syntax Rules and Structure

**Interface tree:**

```typescript
interface TreeNode {
  value: string;
  children: TreeNode[];
}
```

**Generic tree:**

```typescript
type Tree<T> = { value: T; children: Tree<T>[] };
```

**Syntax rules.**

- The `children` property must be an array of the same type.
- Leaf nodes have an empty `children` array.
- Generic trees parameterize the value type.

**Constraints and limitations.**

- Deep trees can hit compiler recursion limits in **type-level** operations, though runtime traversal is fine.
- Cyclic trees (nodes referencing ancestors) cannot be fully typed without `WeakMap` or similar runtime structures.

### Annotated Complete Step by Step Code Examples

#### Example 1: File system tree

```typescript
// example_tree_filesystem.ts
interface FileNode {
  name: string;
  type: "file";
}

interface DirectoryNode {
  name: string;
  type: "directory";
  children: (FileNode | DirectoryNode)[];
}

const root: DirectoryNode = {
  name: "root",
  type: "directory",
  children: [
    { name: "readme.md", type: "file" },
    { name: "src", type: "directory", children: [
      { name: "index.ts", type: "file" },
    ]},
  ],
};

console.log(root.children[1].children[0].name); // Expected: index.ts
```

**Expected Output:**
```
index.ts
```

**Why this output occurs.** `DirectoryNode` recursively contains `FileNode | DirectoryNode` children, allowing nested folders and files. The compiler checks every level.

---

#### Example 2: Generic tree with mapped values

```typescript
// example_tree_generic.ts
type Tree<T> = { value: T; children: Tree<T>[] };

function mapTree<T, U>(tree: Tree<T>, fn: (value: T) => U): Tree<U> {
  return {
    value: fn(tree.value),
    children: tree.children.map((child) => mapTree(child, fn)),
  };
}

const numericTree: Tree<number> = { value: 1, children: [{ value: 2, children: [] }] };
const stringTree = mapTree(numericTree, (n) => `#${n}`);

console.log(stringTree.children[0].value); // Expected: #2
```

**Expected Output:**
```
#2
```

**Why this output occurs.** The generic `Tree<T>` is transformed to `Tree<U>` recursively, preserving the tree shape while changing the value type.

---

### Real-World Cases

- **Abstract syntax trees**: expressions with nested operator and operand nodes.
- **File explorers**: folders containing files and subfolders.
- **Comment threads**: comments with nested replies.
- **Organization charts**: employees with direct reports.

---

## 6. Nested Configurations

### Definitions

**Core Definition.** **Nested configuration types** are recursive object types that model **hierarchical configuration settings**, where sections can contain sub-sections of the same shape.

**Technical Definition.** Nested configurations are typically modeled with recursive mapped types (`DeepPartial`, `DeepReadonly`) or recursive interfaces. They allow **partial updates at any depth** and **immutable configuration trees**. Recursive conditional types apply transformations uniformly across all nesting levels.

**Beginner-Friendly Explanation.** Nested configuration types let you **type deeply nested settings**—like `app.database.connection.pool.size`—without manually defining every level.

### Purposes

- **To model application configuration trees** with nested sections.
- **To support partial updates at any depth** with `DeepPartial`.
- **To enforce immutability at every level** with `DeepReadonly`.
- **To validate configuration shapes** recursively.

### Syntax Rules and Structure

**Recursive interface configuration:**

```typescript
interface Config {
  server: { port: number; host: string };
  database: { connection: { pool: { size: number } } };
}
```

**Recursive transformation:**

```typescript
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;
```

**Syntax rules.**

- Nested configuration types may be defined directly with nested interfaces or derived with recursive utility types.
- `DeepPartial` makes every level optional; `DeepReadonly` makes every level readonly.

**Constraints and limitations.**

- Deeply nested configurations can trigger exponential type expansion with recursive utility types.
- Depth limits or shallow transformations may be necessary for very deep configs.

### Annotated Complete Step by Step Code Examples

#### Example 1: DeepPartial for configuration updates

```typescript
// example_config_deep_partial.ts
interface AppConfig {
  server: { port: number; host: string };
  database: { connection: { pool: { size: number } } };
}

type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;

const update: DeepPartial<AppConfig> = {
  database: { connection: { pool: { size: 10 } } },
};

console.log(update.database?.connection?.pool?.size); // Expected: 10
```

**Expected Output:**
```
10
```

**Why this output occurs.** `DeepPartial` recursively makes every property optional, allowing a partial update at any depth. The optional chaining reflects the possibility of missing intermediate levels.

---

### Real-World Cases

- **Environment configuration**: nested server, database, and logging settings.
- **Theme configuration**: deeply nested color and spacing tokens.
- **Plugin configuration**: recursive plugin settings with nested options.

---

## 7. Recursive Utility Types

### Definitions

**Core Definition.** **Recursive utility types** are generic types that apply a transformation **recursively at every nesting level**, such as `DeepPartial`, `DeepReadonly`, and `DeepRequired`.

**Technical Definition.** Recursive utility types combine **mapped types** with **conditional types** and **self-reference**. `DeepPartial<T>` recursively makes all properties optional; `DeepReadonly<T>` recursively marks all properties readonly; `DeepRequired<T>` recursively removes optionality. The pattern checks arrays before objects, skips functions, and terminates on primitives. Built-in `Awaited<T>` also recursively unwraps promises.

**Beginner-Friendly Explanation.** Recursive utility types are **`Partial`, `Readonly`, and `Required` that work at every level**. They are how you make a deeply nested configuration fully optional, fully readonly, or fully required.

### Purposes

- **To apply a transformation at every nesting level** without manual repetition.
- **To support deep partial updates** in configuration and form state.
- **To enforce deep immutability** in shared state and configuration.
- **To recursively unwrap promises** with `Awaited`.

### Syntax Rules and Structure

**DeepPartial:**

```typescript
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;
```

**DeepReadonly:**

```typescript
type DeepReadonly<T> = T extends (infer U)[]
  ? readonly DeepReadonly<U>[]
  : T extends Function
    ? T
    : T extends object
      ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
      : T;
```

**DeepRequired:**

```typescript
type DeepRequired<T> = T extends object
  ? { [K in keyof T]-?: DeepRequired<T[K]> }
  : T;
```

**Syntax rules.**

- Always include a **base case** for primitives (and often functions and arrays).
- Check arrays before plain objects; functions before objects.
- Use `-?` to remove optionality in `DeepRequired`.
- Consider **depth limits** for deeply nested configurations.

**Constraints and limitations.**

- Unbounded recursive utility types can cause exponential type expansion and compiler hangs.
- Deep recursion beyond ~50 levels triggers “Type instantiation is excessively deep” errors.

### Annotated Complete Step by Step Code Examples

#### Example 1: DeepPartial and DeepReadonly

```typescript
// example_utility_deep.ts
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;

type DeepReadonly<T> = T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;

interface User {
  name: string;
  address: { city: string; zip: string };
}

const partial: DeepPartial<User> = { address: { city: "London" } };
const readonlyUser: DeepReadonly<User> = {
  name: "Ada",
  address: { city: "London", zip: "SW1A" },
};

console.log(partial.address?.city); // Expected: London
// readonlyUser.address.city = "Paris"; // ❌ Error: readonly.
```

**Expected Output:**
```
London
```

**Why this output occurs.** `DeepPartial` makes nested properties optional; `DeepReadonly` marks them readonly. Both recurse through the object structure.

---

#### Example 2: Depth-limited DeepPartial

```typescript
// example_utility_depth_limited.ts
type DeepPartial<T, Depth extends number[] = []> =
  Depth["length"] extends 5
    ? T
    : T extends object
      ? { [K in keyof T]?: DeepPartial<T[K], [...Depth, 1]> }
      : T;

interface DeepConfig {
  a: { b: { c: { d: { e: string } } } };
}

const partial: DeepPartial<DeepConfig> = { a: { b: {} } };
console.log(partial.a?.b); // Expected: {}
```

**Expected Output:**
```
{}
```

**Why this output occurs.** The depth counter stops recursion at five levels, preventing exponential expansion on deeply nested types.

---

### Real-World Cases

- **Configuration updates**: `DeepPartial<AppConfig>` for patch payloads.
- **Immutable state**: `DeepReadonly<State>` for Redux or Zustand stores.
- **Form state**: recursively optional field states.
- **Promise unwrapping**: `Awaited<Promise<Promise<string>>>` → `string`.

---

## 8. Recursive Type Best Practices

### Definitions

**Core Definition.** **Recursive type best practices** are guidelines for writing recursive types that are **correct, terminating, and compiler-friendly**, avoiding exponential expansion, depth-limit errors, and IDE lag.

**Technical Definition.** Best practices center on **explicit depth limits**, **tail-recursive patterns**, **correct base cases**, **avoiding deep recursion in performance-critical paths**, and **preferring shallow built-in utilities** when deep transformation is unnecessary. Recursive types without depth limits can cause exponential type expansion, compilation hangs, and out-of-memory errors. Recursion beyond ~50 levels causes exponential compiler work and “Type instantiation is excessively deep” errors.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep recursive types from freezing the compiler. The big rules are: always add a depth limit or a clear base case; prefer tail recursion; do not use deep recursive utilities on very deep configs; and use shallow built-ins (`Partial`, `Readonly`) when you only need one level.

### Purposes

- **To prevent compiler hangs and exponential type expansion** from unbounded recursion.
- **To avoid “Type instantiation is excessively deep” errors** on deeply nested types.
- **To keep IDE responsiveness** by limiting type computation.
- **To ensure recursive transformations terminate** on primitives, functions, and arrays.
- **To choose the simplest correct type** between shallow and deep transformations.

### Syntax Rules and Structure

Best practices are conventions that interact with recursive type syntax. The core rules are:

1. **Add explicit depth limits.** Use a depth counter (`Depth extends number[] = []`) and stop at a reasonable depth (e.g., 5 or 10) to prevent exponential expansion.
2. **Use tail-recursive patterns.** Structure recursion so the recursive call is the **final result** without additional manipulation; this avoids deep stack growth in the type checker.
3. **Get the base cases right.** Check arrays before objects, functions before objects, and terminate on primitives. Homomorphic mapped types over primitives return the primitive unchanged, which is useful for `DeepPartial`.
4. **Avoid deep recursion in performance-critical paths.** Recursive mapped types like `ExpandRecursively` should be avoided in hot paths unless strictly necessary.
5. **Prefer shallow built-ins when deep transformation is unnecessary.** `Partial<T>`, `Readonly<T>`, and `Required<T>` are cheaper than their deep counterparts.
6. **Flatten or break up complex recursive types.** Break deep recursion into smaller composable aliases and flatten where possible.
7. **Use libraries like `ts-toolbelt` or `type-fest`** for complex recursive utilities that are already optimized and depth-limited.
8. **Test with deeply nested inputs** to ensure the type does not hit instantiation limits.

**Component breakdown (recursive type decision tree):**

```
Do you need to model recursive data?
  ├── Yes → Is the recursion bounded by the data shape (trees, JSON)?
  │        ├── Yes → Use interface or type alias recursion.
  │        └── No → Use a depth-limited recursive conditional type.
  └── No → Do you need a deep transformation?
           ├── Yes → Add a depth limit and correct base cases.
           └── No → Use shallow built-in utilities.
```

**Constraints and limitations.**

- Even depth-limited recursion can be expensive on very wide unions.
- Tail recursion is not always expressible in TypeScript’s conditional type syntax.
- Recursive utility types cannot handle cyclic types without runtime memoization.

### Annotated Complete Step by Step Code Examples

#### Example 1: Depth-limited recursive type

```typescript
// example_best_depth_limit.ts
// ❌ Unbounded recursion can hang the compiler on deep configs.
// type DeepPartial<T> = { [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K] };

// ✅ Depth-limited recursion.
type DeepPartial<T, Depth extends number[] = []> =
  Depth["length"] extends 5
    ? T
    : T extends object
      ? { [K in keyof T]?: DeepPartial<T[K], [...Depth, 1]> }
      : T;

interface Config {
  a: { b: { c: { d: string } } };
}

const partial: DeepPartial<Config> = { a: { b: {} } };
console.log(partial.a?.b); // Expected: {}
```

**Expected Output:**
```
{}
```

**Why this output occurs.** The depth counter stops recursion at five levels, preventing exponential expansion while still allowing meaningful partial updates.

---

#### Example 2: Correct base cases in `DeepReadonly`

```typescript
// example_best_base_cases.ts
type DeepReadonly<T> = T extends (infer U)[]
  ? readonly DeepReadonly<U>[]
  : T extends Function
    ? T
    : T extends object
      ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
      : T;

interface Data {
  list: { id: number }[];
  fn: () => void;
  name: string;
}

const data: DeepReadonly<Data> = {
  list: [{ id: 1 }],
  fn: () => {},
  name: "Ada",
};

// data.list[0].id = 2; // ❌ Error: readonly.
console.log(data.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The conditional branches handle arrays, functions, objects, and primitives in the correct order, ensuring every nested level is transformed without infinite recursion.

---

### Real-World Cases

- **Deep configuration utilities**: depth-limited `DeepPartial` for patch payloads.
- **Immutable state trees**: `DeepReadonly` with correct array and function handling.
- **Type-level libraries**: using `ts-toolbelt` for optimized recursive types.
- **Code review**: rejecting unbounded recursive mapped types in performance-critical code.

---

## References

- Documentation - Recursive Type References (Playground, TS 3.7+) — https://www.typescriptlang.org/play/3-7/types-and-code-flow/recursive-type-references.ts.html
- Documentation - Utility Types (`Awaited`, `Partial`, `Required`) — https://www.typescriptlang.org/docs/handbook/utility-types.html
- Documentation - TypeScript 3.7 Release Notes (Recursive Type Aliases) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html
- Documentation - TypeScript 4.1 Release Notes (Recursive Conditional Types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html
- pproenca/dot-skills - Limit Type Recursion Depth — https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/typescript/references/type-limit-recursion-depth.md
- pproenca/dot-skills - Avoid Deeply Recursive Type Definitions — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/typescript-refactor/references/compile-avoid-deep-recursion.md
- pproenca/dot-skills - Use Recursive Conditional Types for Structural Transformations — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/typescript-advanced-patterns/references/tlp-recursive-conditional-types.md
- Total TypeScript - Use Recursion and Mapped Types to Create a Type Helper — https://www.totaltypescript.com/use-recursion-and-mapped-types-to-create-a-type-helper
- Stack Overflow - Homomorphic Mapped Types and Primitive Termination (DeepPartial) — https://stackoverflow.com/revisions/fd9ce282-ed7c-468c-a8af-917c992bd4b4/view-source
- GeeksforGeeks - Recursive Types and Interfaces in TypeScript (Tree Examples) — https://origin.geeksforgeeks.org/what-are-recursive-types-interfaces-in-typescript/