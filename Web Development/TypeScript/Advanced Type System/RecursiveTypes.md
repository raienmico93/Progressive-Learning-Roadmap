# TypeScript Recursive Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A recursive type is a type that refers to itself within its own definition. This self-reference allows TypeScript to model data structures of arbitrary depth and complexity—trees, linked lists, nested configurations, JSON values, and abstract syntax trees—using a single, elegant type declaration.

**Technical Definition**
In TypeScript, recursive types are expressed through either interfaces or type aliases that include a reference to themselves, directly or indirectly, in one of their member types. Recursive interfaces use the syntax `interface Node { next?: Node }`, while recursive type aliases use `type Tree = { children: Tree[] }`. TypeScript 3.7 removed the previous restriction that prevented type aliases from referencing themselves directly (TypeScript 3.7 Release Notes). Recursive types can involve unions, intersections, mapped types, and conditional types, enabling complex structural transformations. The compiler enforces a recursion depth limit of approximately 50 levels per type instantiation to prevent exponential type-checking work; exceeding this limit produces the error `TS2589: Type instantiation is excessively deep and possibly infinite`. Recursive types are erased at runtime and exist purely at the type level.

**Beginner-Friendly Explanation**
A recursive type is a type that points back to itself. Think of a tree: each branch has smaller branches, and each smaller branch has even smaller branches—the pattern repeats. In TypeScript, you can describe this with a type like `type TreeNode = { value: string; children: TreeNode[] }`. The `children` property is an array of `TreeNode`s, so the type refers to itself. This lets you model any depth of nesting without writing separate types for each level. Recursive types are perfect for linked lists, JSON, file systems, and any data structure where a value can contain other values of the same shape. But be careful: deeply nested recursive types can slow down or even break the TypeScript compiler, so keep them bounded when possible.

### Key Characteristics

- **Self-reference**: The type refers to itself in its definition, directly or indirectly.
- **Interface and alias support**: Both `interface` and `type` declarations support recursion (TypeScript 3.7+ for aliases).
- **Unbounded depth**: Recursive types model data of arbitrary depth (within compiler limits).
- **Compiler depth limit**: TypeScript enforces approximately 50 levels of recursion per instantiation.
- **Conditional recursion**: Recursive types combined with conditional types enable deep transformations like `DeepReadonly` and `DeepPartial`.
- **Runtime erasure**: Recursive types are compile-time only and produce no runtime code.
- **Error TS2589**: Exceeding the depth limit produces "Type instantiation is excessively deep and possibly infinite."

### Prerequisites

- Basic knowledge of TypeScript object types, interfaces, and type aliases
- Familiarity with generics and type parameters
- Understanding of arrays, unions, and intersections
- Basic familiarity with conditional types and mapped types (for recursive utilities)

### Related Programming Areas

- **Data Structures**: Trees, linked lists, graphs, and nested configurations
- **Type-Level Programming**: Recursive conditional types and recursive mapped types
- **Compiler Design**: AST modeling and recursive type checking
- **JSON and Serialization**: Modeling arbitrary JSON values with the `Json` type
- **Utility Types**: `DeepReadonly`, `DeepPartial`, `DeepRequired`, and `Paths<T>` are recursive utilities

### Core Concepts / Features

1. Recursive Type Aliases and Compiler Depth Limits
2. Tree Structures, Linked Lists, and AST Modeling
3. Nested Configuration Parsing and Deep Object Structural Validation
4. Recursive Utility Types (`DeepReadonly<T>`, `DeepPartial<T>`)


## 1. Recursive Type Aliases and Compiler Depth Limits

### Definitions

**Core Definition**
A recursive type alias is a `type` declaration that references itself in its definition. TypeScript 3.7 lifted the previous restriction that prevented type aliases from directly referring to themselves, enabling more expressive recursive type definitions using unions, intersections, and conditional types. However, the compiler enforces a depth limit to prevent infinite type expansion.

**Technical Definition**
Recursive type aliases can reference themselves directly (e.g., `type Json = string | Json[] | { [key: string]: Json }`) or indirectly through other type aliases. The TypeScript compiler imposes an implicit recursion limit of approximately 50 levels per type instantiation. When this limit is exceeded, the compiler produces the error `TS2589: Type instantiation is excessively deep and possibly infinite`. This limit exists to prevent exponential type-checking work and maintain reasonable compilation times. Recursive types that nest beyond ~50 levels cause exponential compiler work, IDE lag, and cryptic errors. The limit applies per instantiation, meaning multiple deeply nested types in a compilation unit compete for the same budget.

**Beginner-Friendly Explanation**
Recursive type aliases let you define types that refer to themselves using the `type` keyword. Before TypeScript 3.7, you couldn't do this directly—you had to use an interface as a workaround. Now you can write things like `type Json = string | number | Json[] | { [key: string]: Json }`. But there's a catch: TypeScript limits how deeply it will recurse when checking these types. The limit is about 50 levels. If your recursive type goes deeper than that, TypeScript gives up with a "Type instantiation is excessively deep" error. This is why you should add explicit depth limits to recursive utilities—it tells TypeScript when to stop recursing and prevents compiler errors.

### Purposes

- To model data structures with self-referential definitions (JSON, trees, linked lists).
- To create recursive utility types that transform nested structures.
- To overcome the historical limitation of non-recursive type aliases.
- To enable expressive type-level algorithms with union and conditional recursion.
- To understand and avoid compiler depth limit errors (TS2589).

### Syntax Rules and Structure

**General Syntax: Direct Recursive Type Alias**

```typescript
type Json =
  | string
  | number
  | boolean
  | null
  | Json[]
  | { [key: string]: Json };
```

**Component Breakdown**
- `Json` references itself in the array and object alternatives.
- The recursion is guarded by array and object types.

**General Syntax: Indirect Recursive Type Alias**

```typescript
type ValueOrArray<T> = T | Array<ValueOrArray<T>>;
```

**Component Breakdown**
- `ValueOrArray<T>` references itself through `Array<ValueOrArray<T>>`.
- Works in TypeScript 3.7+.

**General Syntax: Depth-Limited Recursion**

```typescript
type I = 0 | 1 | 2 | 3 | 4 | 5;
type Iterate<A extends I = 0> = A extends 0 ? 1 : A extends 1 ? 2 : /* ... */ 5;

type Paths<Obj, X extends I = 0> =
  Iterate<X> extends 5
    ? []
    : Obj extends object
    ? { [Key in keyof Obj]: Prepend<Paths<Obj[Key], Iterate<X>>, Key> }[keyof Obj]
    : [];
```

**Component Breakdown**
- `Iterate<X>` increments the depth counter.
- `Iterate<X> extends 5 ? []` stops recursion at depth 5.
- This pattern explicitly bounds recursion to avoid TS2589.

**Syntax Rules**

- Recursive type aliases can reference themselves directly (TypeScript 3.7+).
- The self-reference must be "guarded" by an object type, array type, tuple type, or conditional type.
- Direct self-reference (`type X = X`) is still an error.
- The compiler enforces a recursion limit of approximately 50 levels.
- Exceeding the limit produces `TS2589: Type instantiation is excessively deep and possibly infinite`.
- Use a depth counter (`Iterate<X>`) to bound recursion explicitly.
- Flatten recursive types where possible to reduce depth.

**Constraints and Limitations**

- The 50-level recursion limit is a hard constraint (increased from 50 to 500 in TypeScript 4.5, but still finite).
- Recursive types that nest beyond ~50 levels cause exponential compiler work.
- Deeply nested recursive types can cause IDE lag and slow type checking.
- Error messages for TS2589 are often cryptic and difficult to diagnose.
- The limit applies per instantiation, so multiple deep types compete for budget.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Recursive Type Alias

```typescript
// Step 1: Define a recursive JSON type.
type Json =
  | string
  | number
  | boolean
  | null
  | Json[]
  | { [key: string]: Json };

// Step 2: Create JSON values of arbitrary depth.
const data: Json = {
  name: "Alice",
  age: 30,
  hobbies: ["reading", "coding"],
  address: {
    street: "123 Main St",
    city: "Springfield",
    coordinates: { lat: 39.78, lng: -89.65 },
  },
};

console.log(data);  // { name: 'Alice', age: 30, ... }

// Step 3: Recursive processing.
function countKeys(value: Json): number {
  if (typeof value !== "object" || value === null) return 0;
  if (Array.isArray(value)) {
    return value.reduce((sum, item) => sum + countKeys(item), 0);
  }
  return Object.keys(value).length +
    Object.values(value).reduce((sum, v) => sum + countKeys(v), 0);
}

console.log(countKeys(data));  // 9

// Step 4: The type works at arbitrary depth (within compiler limits).
const deep: Json = { a: { b: { c: { d: { e: [1, 2, 3] } } } } };
console.log(countKeys(deep));  // 8

// Step 5: Invalid values are caught.
// const invalid: Json = { fn: () => {} };
// ❌ Error: Function is not assignable to Json.
```

**Expected Output:**
```
{ name: 'Alice', age: 30, ... }
9
8
```

**Why This Output Occurs:** The `Json` type recursively describes all valid JSON values. The `countKeys` function recursively traverses the structure. The type works at arbitrary depth until the compiler's recursion limit is reached.

#### Example 2: Depth-Limited Recursion to Avoid TS2589

```typescript
// Step 1: Define a depth counter.
type Depth = 0 | 1 | 2 | 3 | 4 | 5;
type Increment<D extends Depth> =
  D extends 0 ? 1 : D extends 1 ? 2 : D extends 2 ? 3 :
  D extends 3 ? 4 : D extends 4 ? 5 : 5;

// Step 2: Define a depth-limited recursive type.
type DeepReadonlyLimited<T, D extends Depth = 0> =
  D extends 5
    ? T  // Stop at depth 5
    : T extends object
    ? { readonly [K in keyof T]: DeepReadonlyLimited<T[K], Increment<D>> }
    : T;

// Step 3: Use the depth-limited type.
interface Config {
  server: {
    host: string;
    port: number;
    ssl: { enabled: boolean; cert: string };
  };
}

type ReadonlyConfig = DeepReadonlyLimited<Config>;
// All properties at all levels are readonly (up to depth 5).

const config: ReadonlyConfig = {
  server: {
    host: "localhost",
    port: 3000,
    ssl: { enabled: true, cert: "cert.pem" },
  },
};

// config.server.host = "other";  // ❌ Error: readonly.
console.log(config.server.host);  // "localhost"

// Step 4: Without depth limiting, deeply nested types would hit TS2589.
// The depth counter prevents this by stopping recursion at depth 5.
console.log("Depth-limited recursion complete.");
```

**Expected Output:**
```
localhost
Depth-limited recursion complete.
```

**Why This Output Occurs:** The `DeepReadonlyLimited<T, D>` type uses a depth counter `D` to limit recursion. When `D` reaches 5, the recursion stops and returns `T` unchanged. This prevents the `TS2589` error that would occur with unbounded recursion.

### Real-World Cases

**Case 1: JSON Parsing**
The `Json` recursive type is used to type `JSON.parse()` results, forcing validation before accessing nested properties.

**Case 2: File System Modeling**
File systems are naturally recursive: a directory contains files and subdirectories, each of which is itself a directory or file. Recursive types model this hierarchy.

**Case 3: Organizational Charts**
Organizational hierarchies (CEO → managers → employees) are recursive structures that use recursive types for modeling reporting relationships.

**Case 4: Comment Threads**
Nested comment threads (comments with replies, which have replies) are recursive structures modeled with recursive types.

**Case 5: Component Trees**
React and Vue component trees are recursive: a component can render child components, which can render their own children.

### References

- TypeScript 3.7 Release Notes: Recursive Type Aliases — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html
- TypeScript Playground: Recursive Type References — https://www.typescriptlang.org/play/3-7/types-and-code-flow/recursive-type-references.ts.html
- GeeksforGeeks: What are Recursive Types & Interfaces in TypeScript? — https://origin.geeksforgeeks.org/what-are-recursive-types-interfaces-in-typescript/
- FreeCodeCamp: Recursive Types in TypeScript — https://www.freecodecamp.org/news/recursive-types-in-typescript-a-brief-exploration/
- Stack Overflow: Using recursive type alias in generic results in error — https://stackoverflow.com/questions/60265325
- GitHub Issue #30188: Recursion limit and type instantiation — https://github.com/microsoft/TypeScript/issues/30188
- GitHub PR #45025: Increase type instantiation depth limit from 50 to 500 — https://github.com/microsoft/TypeScript/pull/45025
- GeeksforGeeks: What is Recursive Generic in TypeScript? — https://origin.geeksforgeeks.org/what-is-recursive-generic-in-typescript/


## 2. Tree Structures, Linked Lists, and AST Modeling

### Definitions

**Core Definition**
Tree structures, linked lists, and abstract syntax trees (ASTs) are the canonical data structures modeled by recursive types. A tree has nodes with child nodes of the same type; a linked list has nodes with a reference to the next node of the same type; an AST has nodes that contain child expressions or statements of the same recursive type.

**Technical Definition**
Recursive types model these structures by including a self-reference in the type definition. For trees, the pattern is `type Tree<T> = { value: T; children: Tree<T>[] }`. For linked lists, `type ListNode<T> = { value: T; next: ListNode<T> | null }`. For ASTs, the pattern involves discriminated unions where each node kind has children that are themselves AST nodes: `type Expr = NumberLiteral | BinaryExpression`, where `BinaryExpression` has `left: Expr` and `right: Expr`. The self-reference can be optional (`next?: ListNode`), nullable (`next: ListNode | null`), or an array (`children: TreeNode[]`). Generic type parameters enable trees and lists of any value type.

**Beginner-Friendly Explanation**
Trees, linked lists, and ASTs are all "self-similar" structures—each part looks like the whole. A tree branch is itself a smaller tree. A linked list node points to the next node, which is itself a linked list. An AST expression contains sub-expressions that are themselves AST expressions. Recursive types are perfect for these because they let you say "a tree is a value plus a list of trees" or "a list node is a value plus the next node." TypeScript then understands the structure at any depth, giving you type-safe access to nested nodes, children, and sub-expressions.

### Purposes

- To model hierarchical data structures (trees, file systems, organizational charts).
- To model sequential data structures with dynamic length (linked lists, queues).
- To model programming language syntax (ASTs for compilers, interpreters, linters).
- To enable recursive algorithms (tree traversal, list reversal, AST transformation).
- To provide type safety for nested data structures at any depth.

### Syntax Rules and Structure

**General Syntax: Tree Node**

```typescript
type TreeNode<T> = {
  value: T;
  children: TreeNode<T>[];
};
```

**Component Breakdown**
- `value: T`: The node's value.
- `children: TreeNode<T>[]`: An array of child nodes of the same type.

**General Syntax: Linked List Node**

```typescript
type ListNode<T> = {
  value: T;
  next: ListNode<T> | null;
};
```

**Component Breakdown**
- `next: ListNode<T> | null`: A reference to the next node or `null`.

**General Syntax: AST Node (Discriminated Union)**

```typescript
type Expr =
  | { kind: "number"; value: number }
  | { kind: "binary"; left: Expr; right: Expr; operator: string };
```

**Component Breakdown**
- `left: Expr; right: Expr`: Child expressions of the same recursive type.
- The `kind` discriminant enables narrowing.

**General Syntax: Binary Tree**

```typescript
type BinaryTree<T> = {
  value: T;
  left: BinaryTree<T> | null;
  right: BinaryTree<T> | null;
};
```

**Component Breakdown**
- `left` and `right` are either subtrees or `null`.

**Syntax Rules**

- Tree nodes use an array of children: `children: TreeNode<T>[]`.
- Linked list nodes use a nullable next reference: `next: ListNode<T> | null`.
- AST nodes use discriminated unions with recursive children.
- Generic type parameters enable trees/lists of any value type.
- Optional references (`next?`) or nullable references (`next | null`) terminate the recursion.
- Recursive algorithms use the same structure: base case at leaf/null, recursive case at internal nodes.

**Constraints and Limitations**

- Deeply nested trees/lists can hit the compiler recursion limit (TS2589).
- Type inference for recursive types requires explicit annotation (TypeScript cannot infer recursive types from object literals).
- Runtime recursion depth is limited by the JavaScript call stack (separate from the compiler limit).
- Tree traversal algorithms must handle the base case (leaf nodes) to avoid infinite recursion.
- AST types with many node kinds can produce large unions.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Tree Structure

```typescript
// Step 1: Define a recursive tree type.
type TreeNode<T> = {
  value: T;
  children: TreeNode<T>[];
};

// Step 2: Create a tree.
const tree: TreeNode<string> = {
  value: "root",
  children: [
    {
      value: "child1",
      children: [{ value: "grandchild1", children: [] }],
    },
    { value: "child2", children: [] },
  ],
};

// Step 3: Recursive traversal.
function printTree(node: TreeNode<string>, depth: number = 0): void {
  const indent = "  ".repeat(depth);
  console.log(`${indent}${node.value}`);
  node.children.forEach((child) => printTree(child, depth + 1));
}

printTree(tree);
// "root"
// "  child1"
// "    grandchild1"
//   "child2"

// Step 4: Recursive depth calculation.
function treeDepth(node: TreeNode<string>): number {
  if (node.children.length === 0) return 1;
  return 1 + Math.max(...node.children.map(treeDepth));
}

console.log(`Depth: ${treeDepth(tree)}`);  // "Depth: 3"

// Step 5: Recursive search.
function findNode(node: TreeNode<string>, target: string): TreeNode<string> | null {
  if (node.value === target) return node;
  for (const child of node.children) {
    const found = findNode(child, target);
    if (found) return found;
  }
  return null;
}

const found = findNode(tree, "grandchild1");
console.log(found?.value);  // "grandchild1"
```

**Expected Output:**
```
root
  child1
    grandchild1
  child2
Depth: 3
grandchild1
```

**Why This Output Occurs:** The `TreeNode<T>` type recursively models a tree. The `printTree` function recursively traverses the tree, indenting by depth. The `treeDepth` function recursively calculates the depth. The `findNode` function recursively searches for a node.

#### Example 2: Linked List

```typescript
// Step 1: Define a recursive linked list type.
type ListNode<T> = {
  value: T;
  next: ListNode<T> | null;
};

// Step 2: Create a linked list.
const list: ListNode<number> = {
  value: 1,
  next: {
    value: 2,
    next: {
      value: 3,
      next: null,
    },
  },
};

// Step 3: Recursive traversal.
function printList(node: ListNode<number> | null): void {
  if (node === null) return;
  console.log(node.value);
  printList(node.next);
}

printList(list);  // 1, 2, 3

// Step 4: Recursive length calculation.
function listLength(node: ListNode<number> | null): number {
  if (node === null) return 0;
  return 1 + listLength(node.next);
}

console.log(`Length: ${listLength(list)}`);  // "Length: 3"

// Step 5: Recursive reversal.
function reverse<T>(head: ListNode<T> | null): ListNode<T> | null {
  let prev: ListNode<T> | null = null;
  let current = head;
  while (current !== null) {
    const next = current.next;
    current.next = prev;
    prev = current;
    current = next;
  }
  return prev;
}

const reversed = reverse(list);
printList(reversed);  // 3, 2, 1
```

**Expected Output:**
```
1
2
3
Length: 3
3
2
1
```

**Why This Output Occurs:** The `ListNode<T>` type recursively models a linked list. The `printList` function recursively traverses the list. The `listLength` function recursively counts nodes. The `reverse` function reverses the list iteratively (to avoid deep recursion).

#### Example 3: AST Modeling

```typescript
// Step 1: Define a recursive AST type.
type Expr =
  | { kind: "number"; value: number }
  | { kind: "string"; value: string }
  | { kind: "binary"; left: Expr; right: Expr; operator: string }
  | { kind: "unary"; operand: Expr; operator: string };

// Step 2: Create an AST for the expression (1 + 2) * -3.
const ast: Expr = {
  kind: "binary",
  operator: "*",
  left: {
    kind: "binary",
    operator: "+",
    left: { kind: "number", value: 1 },
    right: { kind: "number", value: 2 },
  },
  right: {
    kind: "unary",
    operator: "-",
    operand: { kind: "number", value: 3 },
  },
};

// Step 3: Recursive evaluation.
function evaluate(expr: Expr): number {
  switch (expr.kind) {
    case "number": return expr.value;
    case "string": throw new Error("Cannot evaluate string");
    case "binary": {
      const left = evaluate(expr.left);
      const right = evaluate(expr.right);
      switch (expr.operator) {
        case "+": return left + right;
        case "-": return left - right;
        case "*": return left * right;
        case "/": return left / right;
      }
    }
    case "unary": {
      const operand = evaluate(expr.operand);
      if (expr.operator === "-") return -operand;
      return operand;
    }
  }
}

console.log(evaluate(ast));  // -9

// Step 4: Recursive stringification.
function stringify(expr: Expr): string {
  switch (expr.kind) {
    case "number": return String(expr.value);
    case "string": return `"${expr.value}"`;
    case "binary": return `(${stringify(expr.left)} ${expr.operator} ${stringify(expr.right)})`;
    case "unary": return `${expr.operator}${stringify(expr.operand)}`;
  }
}

console.log(stringify(ast));  // "((1 + 2) * -3)"
```

**Expected Output:**
```
-9
((1 + 2) * -3)
```

**Why This Output Occurs:** The `Expr` type recursively models an AST. The `evaluate` function recursively evaluates the expression. The `stringify` function recursively builds a string representation. The discriminated union (`kind`) enables exhaustive handling of all node types.

### Real-World Cases

**Case 1: File System Libraries**
File system APIs use recursive types to model directories and files, enabling recursive traversal and operations.

**Case 2: Comment Systems**
Nested comment threads use recursive types to model replies to replies, enabling recursive rendering and counting.

**Case 3: Compiler Frontends**
TypeScript, Babel, and ESLint use recursive AST types to represent parsed code, enabling recursive traversal and transformation.

**Case 4: DOM Trees**
DOM nodes are recursive: each element can contain child elements. Recursive types model the DOM hierarchy.

**Case 5: JSON and Configuration**
JSON values and nested configuration objects are naturally recursive and are modeled with recursive types.

### References

- FreeCodeCamp: Recursive Types in TypeScript — https://www.freecodecamp.org/news/recursive-types-in-typescript-a-brief-exploration/
- GeeksforGeeks: What are Recursive Types & Interfaces in TypeScript? — https://origin.geeksforgeeks.org/what-are-recursive-types-interfaces-in-typescript/
- GeeksforGeeks: What is Recursive Generic in TypeScript? — https://origin.geeksforgeeks.org/what-is-recursive-generic-in-typescript/
- TypeScript Playground: Recursive Type References — https://www.typescriptlang.org/play/3-7/types-and-code-flow/recursive-type-references.ts.html
- Stack Overflow: Iterate Over TypeScript Type-level Linked List without Excessive Depth Error — https://stackoverflow.com/questions/64912139


## 3. Nested Configuration Parsing and Deep Object Structural Validation

### Definitions

**Core Definition**
Nested configuration parsing uses recursive types to model configuration objects with arbitrary depth. Deep object structural validation uses recursive types to validate that nested data conforms to a required structure, enabling type-safe processing of configuration files, API responses, and user input.

**Technical Definition**
Nested configuration types use recursive type aliases or interfaces to describe configuration schemas where sections can contain subsections. For example, `type Config = { [key: string]: Config | string | number }`. Deep object validation combines recursive types with conditional types to enforce structural constraints at every level of nesting. TypeScript enforces a recursion limit of approximately 50 levels per type instantiation when validating deeply nested structures. When multiple deeply nested types exist in a compilation unit, they compete for this shared budget, and deeply nested structures are automatically extracted into separate named validators to prevent type depth issues. Recursive validation can also be achieved through runtime schema validation libraries (Zod, Yup) that mirror the recursive types.

**Beginner-Friendly Explanation**
Nested configuration is when a config object has sections that contain more sections, like a tree of settings. Recursive types let you describe this: "a config is an object where each value is either a config, a string, or a number." Deep validation means checking that a config object conforms to this structure at every level. TypeScript can do this at the type level using recursive conditional types, but there's a limit—about 50 levels deep. Beyond that, the compiler gives up with a "Type instantiation is excessively deep" error. For very deep configs, you might need to flatten the structure or use a depth limiter. Runtime schema validation libraries like Zod can also validate nested configs, and their types can be derived from the schema.

### Purposes

- To model configuration objects with arbitrary nesting depth.
- To validate that nested data conforms to a required structure at every level.
- To enable type-safe access to deeply nested configuration values.
- To prevent runtime errors from accessing invalid configuration paths.
- To combine compile-time type checking with runtime validation for external data.

### Syntax Rules and Structure

**General Syntax: Recursive Configuration Type**

```typescript
type ConfigValue = string | number | boolean | ConfigValue[] | { [key: string]: ConfigValue };

type Config = {
  [key: string]: ConfigValue;
};
```

**Component Breakdown**
- `ConfigValue` recursively includes objects and arrays.
- Any key can map to a primitive, array, or nested object.

**General Syntax: Structural Validation with Conditional Types**

```typescript
type ValidateConfig<T> =
  T extends object
    ? { [K in keyof T]: ValidateConfig<T[K]> }
    : T extends string | number | boolean
    ? T
    : never;
```

**Component Breakdown**
- Recursively validates that each level matches the expected structure.
- Invalid types produce `never`.

**General Syntax: Depth-Limited Validation**

```typescript
type Depth = 0 | 1 | 2 | 3 | 4 | 5;

type ValidateLimited<T, D extends Depth = 0> =
  D extends 5
    ? T
    : T extends object
    ? { [K in keyof T]: ValidateLimited<T[K], Increment<D>> }
    : T extends string | number | boolean
    ? T
    : never;
```

**Component Breakdown**
- Stops validation at depth 5 to avoid TS2589.

**Syntax Rules**

- Recursive configuration types use `[key: string]: ConfigValue` for dynamic keys.
- Config values can be primitives, arrays, or nested objects.
- Deep validation uses recursive conditional types.
- Depth limiting prevents TS2589 errors.
- TypeScript's recursion limit is approximately 50 levels.
- Runtime validation (Zod, Yup) complements compile-time types for external data.

**Constraints and Limitations**

- The 50-level recursion limit prevents validation of very deep structures.
- Deeply nested types compete for the shared recursion budget.
- TypeScript cannot infer recursive types from object literals—explicit annotation is required.
- Runtime validation is still needed for external data (APIs, user input).
- Recursive types with many properties can slow down the compiler.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Recursive Configuration Type

```typescript
// Step 1: Define a recursive configuration type.
type ConfigValue = string | number | boolean | ConfigValue[] | { [key: string]: ConfigValue };

type AppConfig = {
  [key: string]: ConfigValue;
};

// Step 2: Create a nested configuration object.
const config: AppConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  features: {
    darkMode: true,
    notifications: {
      email: { enabled: true, frequency: "daily" },
      push: { enabled: false },
    },
  },
  retry: { maxAttempts: 3, backoff: "exponential" },
};

// Step 3: Type-safe access to nested values.
console.log(config.apiUrl);  // "https://api.example.com"
console.log((config.features as any).notifications.email.frequency);  // "daily"

// Step 4: Recursive traversal.
function printConfig(obj: AppConfig, prefix: string = ""): void {
  for (const [key, value] of Object.entries(obj)) {
    const path = prefix ? `${prefix}.${key}` : key;
    if (typeof value === "object" && value !== null && !Array.isArray(value)) {
      printConfig(value as AppConfig, path);
    } else {
      console.log(`${path}: ${JSON.stringify(value)}`);
    }
  }
}

printConfig(config);
// "apiUrl: "https://api.example.com""
// "timeout: 5000"
// "features.darkMode: true"
// "features.notifications.email.enabled: true"
// ...

console.log("Config traversal complete.");
```

**Expected Output:**
```
https://api.example.com
daily
apiUrl: "https://api.example.com"
timeout: 5000
features.darkMode: true
features.notifications.email.enabled: true
...
Config traversal complete.
```

**Why This Output Occurs:** The `AppConfig` type recursively describes nested configuration. The `printConfig` function recursively traverses the configuration, handling nested objects and arrays. The type allows arbitrary depth within compiler limits.

#### Example 2: Depth-Limited Structural Validation

```typescript
// Step 1: Define a depth counter.
type Depth = 0 | 1 | 2 | 3 | 4 | 5;
type Increment<D extends Depth> =
  D extends 0 ? 1 : D extends 1 ? 2 : D extends 2 ? 3 :
  D extends 3 ? 4 : D extends 4 ? 5 : 5;

// Step 2: Define a depth-limited validation type.
type Validate<T, D extends Depth = 0> =
  D extends 5
    ? T
    : T extends string | number | boolean | null
    ? T
    : T extends Array<infer U>
    ? Validate<U, Increment<D>>[]
    : T extends object
    ? { [K in keyof T]: Validate<T[K], Increment<D>> }
    : never;

// Step 3: Use the validation type.
interface RawConfig {
  server: {
    host: string;
    port: number;
    ssl: { enabled: boolean; cert: string };
  };
  database: {
    url: string;
    credentials: { username: string; password: string };
  };
}

type ValidatedConfig = Validate<RawConfig>;
// All levels are validated (up to depth 5).

const config: ValidatedConfig = {
  server: {
    host: "localhost",
    port: 3000,
    ssl: { enabled: true, cert: "cert.pem" },
  },
  database: {
    url: "postgres://localhost",
    credentials: { username: "admin", password: "secret" },
  },
};

console.log(config.server.host);  // "localhost"

// Step 4: Invalid nested types produce never.
// type Invalid = Validate<{ a: { b: () => void } }>;
// The function type at depth 2 produces never.

console.log("Depth-limited validation complete.");
```

**Expected Output:**
```
localhost
Depth-limited validation complete.
```

**Why This Output Occurs:** The `Validate<T, D>` type recursively validates each level of the configuration, stopping at depth 5. Invalid types (like functions) produce `never`, effectively rejecting the type. The depth counter prevents TS2589 errors.

### Real-World Cases

**Case 1: Application Configuration**
Applications with deeply nested configuration (server, database, features, logging) use recursive types to model and validate configuration structure.

**Case 2: JSON Schema Validation**
JSON Schema definitions are recursive; libraries like `json-schema-to-ts` use recursive types to derive TypeScript types from schemas.

**Case 3: API Response Validation**
API responses with nested data structures use recursive types combined with runtime validation (Zod, Yup) to ensure data integrity.

**Case 4: Form Data Validation**
Nested form data (address with street, city, zip; contact with email, phone) uses recursive validation types.

**Case 5: Environment Configuration**
Environment configuration objects (nested env vars) use recursive types for type-safe access and validation.

### References

- FreeCodeCamp: Recursive Types in TypeScript — https://www.freecodecamp.org/news/recursive-types-in-typescript-a-brief-exploration/
- GeeksforGeeks: What are Recursive Types & Interfaces in TypeScript? — https://origin.geeksforgeeks.org/what-are-recursive-types-interfaces-in-typescript/
- Stack Overflow: Using recursive type alias in generic results in error — https://stackoverflow.com/questions/60265325
- TS2589 Usage Guide: Preventing "Type instantiation excessively deep" — https://raw.githubusercontent.com/libar-dev/zod-convex-packages/main/docs/ts2589-usage.md
- TypeScript Playground: Recursive Type References — https://www.typescriptlang.org/play/3-7/types-and-code-flow/recursive-type-references.ts.html


## 4. Recursive Utility Types (`DeepReadonly<T>`, `DeepPartial<T>`)

### Definitions

**Core Definition**
Recursive utility types are generic type transformers that apply an operation at every level of a nested structure. `DeepReadonly<T>` makes all properties, at all levels, readonly; `DeepPartial<T>` makes all properties, at all levels, optional. They extend TypeScript's standard shallow utility types (`Readonly<T>`, `Partial<T>`) to work recursively on nested objects and arrays.

**Technical Definition**
Recursive utility types are implemented using recursive conditional types combined with mapped types. The pattern is: if `T` is a primitive, return `T` unchanged; if `T` is an array, recursively apply the transformation to the element type; if `T` is an object, recursively apply the transformation to each property. The recursion is guarded by object and array type checks. Depth limiting with an iteration counter prevents `TS2589` errors in deeply nested structures. Libraries like `ts-deep-types` and `utility-types` provide pre-built versions of these utilities. The TypeScript standard library includes `Awaited<T>` (recursive promise unwrapping) but not deep object utilities.

**Beginner-Friendly Explanation**
TypeScript's built-in `Readonly<T>` and `Partial<T>` only work on the top level of an object. If you have a nested config and want to make *everything* readonly, you need `DeepReadonly<T>`. It recursively walks through the object, making every property readonly at every level. Same for `DeepPartial<T>`—it makes every property optional, at every level, which is perfect for deep updates. These utilities are built using recursion: the type checks if it's an object, and if so, applies itself to each property. You can add a depth counter to stop recursion before hitting TypeScript's limit.

### Purposes

- To make all properties at all levels readonly (`DeepReadonly<T>`).
- To make all properties at all levels optional (`DeepPartial<T>`).
- To create deep immutable state types for Redux, Zustand, and other state managers.
- To enable deep update patterns where any nested field can be partially updated.
- To provide recursive transformations that the standard library lacks.

### Syntax Rules and Structure

**General Syntax: `DeepReadonly<T>`**

```typescript
type DeepReadonly<T> =
  T extends (infer U)[]
    ? ReadonlyArray<DeepReadonly<U>>
    : T extends object
    ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
    : T;
```

**Component Breakdown**
- Arrays: `ReadonlyArray<DeepReadonly<U>>` makes array elements readonly.
- Objects: `{ readonly [K in keyof T]: DeepReadonly<T[K]> }` makes all properties readonly recursively.
- Primitives: Returned unchanged.

**General Syntax: `DeepPartial<T>`**

```typescript
type DeepPartial<T> =
  T extends (infer U)[]
    ? DeepPartial<U>[]
    : T extends object
    ? { [K in keyof T]?: DeepPartial<T[K]> }
    : T;
```

**Component Breakdown**
- Objects: `{ [K in keyof T]?: DeepPartial<T[K]> }` makes all properties optional recursively.
- Arrays: `DeepPartial<U>[]` makes array elements deeply partial.

**General Syntax: Depth-Limited `DeepReadonly<T>`**

```typescript
type Depth = 0 | 1 | 2 | 3 | 4 | 5;
type Increment<D extends Depth> = D extends 0 ? 1 : /* ... */ 5;

type DeepReadonly<T, D extends Depth = 0> =
  D extends 5
    ? T
    : T extends (infer U)[]
    ? ReadonlyArray<DeepReadonly<U, Increment<D>>>
    : T extends object
    ? { readonly [K in keyof T]: DeepReadonly<T[K], Increment<D>> }
    : T;
```

**Component Breakdown**
- `D extends 5 ? T` stops recursion at depth 5.

**Syntax Rules**

- The recursive utility checks for arrays first (`T extends (infer U)[]`).
- Then checks for objects (`T extends object`).
- Primitives are returned unchanged (base case).
- Depth limiting prevents TS2589 errors.
- `DeepReadonly` uses `ReadonlyArray` for arrays.
- `DeepPartial` uses `?` for optionality.
- These utilities are implemented as recursive conditional types.

**Constraints and Limitations**

- Unbounded recursion can hit TS2589 (approximately 50 levels).
- Depth limiting adds complexity but prevents compiler errors.
- The utilities are compile-time only; no runtime transformation occurs.
- Very deep structures (100+ levels) cannot be fully transformed.
- Recursive utility types can be slow to type-check for large structures.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `DeepReadonly<T>`

```typescript
// Step 1: Define DeepReadonly.
type DeepReadonly<T> =
  T extends (infer U)[]
    ? ReadonlyArray<DeepReadonly<U>>
    : T extends object
    ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
    : T;

// Step 2: Define a nested configuration type.
interface Config {
  server: {
    host: string;
    port: number;
    ssl: { enabled: boolean; cert: string };
  };
  features: string[];
}

// Step 3: Apply DeepReadonly.
type ReadonlyConfig = DeepReadonly<Config>;

const config: ReadonlyConfig = {
  server: {
    host: "localhost",
    port: 3000,
    ssl: { enabled: true, cert: "cert.pem" },
  },
  features: ["auth", "logging"],
};

// Step 4: All levels are readonly.
// config.server.host = "other";        // ❌ Error: readonly.
// config.server.ssl.enabled = false;   // ❌ Error: readonly.
// config.features.push("metrics");     // ❌ Error: readonly array.

console.log(config.server.host);      // "localhost"
console.log(config.features[0]);      // "auth"

// Step 5: Nested access is type-safe.
const cert: string = config.server.ssl.cert;
console.log(cert);  // "cert.pem"
```

**Expected Output:**
```
localhost
auth
cert.pem
```

**Why This Output Occurs:** `DeepReadonly<T>` recursively makes every property readonly at every level. Arrays become `ReadonlyArray`, preventing mutation. The type ensures immutability throughout the nested structure.

#### Example 2: `DeepPartial<T>`

```typescript
// Step 1: Define DeepPartial.
type DeepPartial<T> =
  T extends (infer U)[]
    ? DeepPartial<U>[]
    : T extends object
    ? { [K in keyof T]?: DeepPartial<T[K]> }
    : T;

// Step 2: Define a nested configuration type.
interface Config {
  server: {
    host: string;
    port: number;
    ssl: { enabled: boolean; cert: string };
  };
  database: {
    url: string;
    credentials: { username: string; password: string };
  };
}

// Step 3: Apply DeepPartial.
type PartialConfig = DeepPartial<Config>;

// Step 4: Partial updates at any depth.
const update1: PartialConfig = { server: { host: "newhost" } };
const update2: PartialConfig = { database: { credentials: { username: "admin" } } };
const update3: PartialConfig = {};

console.log(update1);  // { server: { host: 'newhost' } }
console.log(update2);  // { database: { credentials: { username: 'admin' } } }

// Step 5: Deep merge function.
function deepMerge<T>(base: T, patch: DeepPartial<T>): T {
  const result = { ...base } as T;
  for (const key in patch) {
    const patchValue = patch[key];
    if (patchValue !== undefined) {
      if (typeof patchValue === "object" && patchValue !== null && !Array.isArray(patchValue)) {
        (result as any)[key] = deepMerge((base as any)[key] ?? {}, patchValue);
      } else {
        (result as any)[key] = patchValue;
      }
    }
  }
  return result;
}

const baseConfig: Config = {
  server: { host: "localhost", port: 3000, ssl: { enabled: false, cert: "" } },
  database: { url: "postgres://localhost", credentials: { username: "admin", password: "" } },
};

const merged = deepMerge(baseConfig, { server: { ssl: { enabled: true } } });
console.log(merged.server.ssl.enabled);  // true
console.log(merged.server.host);         // "localhost" (unchanged)
```

**Expected Output:**
```
{ server: { host: 'newhost' } }
{ database: { credentials: { username: 'admin' } } }
true
localhost
```

**Why This Output Occurs:** `DeepPartial<T>` makes every property at every level optional. The `deepMerge` function uses this type to accept partial updates at any depth, recursively merging objects while preserving unchanged values.

#### Example 3: Depth-Limited `DeepReadonly<T>`

```typescript
// Step 1: Define a depth counter.
type Depth = 0 | 1 | 2 | 3 | 4 | 5;
type Increment<D extends Depth> =
  D extends 0 ? 1 : D extends 1 ? 2 : D extends 2 ? 3 :
  D extends 3 ? 4 : D extends 4 ? 5 : 5;

// Step 2: Define depth-limited DeepReadonly.
type DeepReadonlyLimited<T, D extends Depth = 0> =
  D extends 5
    ? T
    : T extends (infer U)[]
    ? ReadonlyArray<DeepReadonlyLimited<U, Increment<D>>>
    : T extends object
    ? { readonly [K in keyof T]: DeepReadonlyLimited<T[K], Increment<D>> }
    : T;

// Step 3: Use the depth-limited type.
interface DeepConfig {
  level1: {
    level2: {
      level3: {
        level4: {
          level5: { value: string };
        };
      };
    };
  };
}

type ReadonlyDeepConfig = DeepReadonlyLimited<DeepConfig>;
// All levels up to depth 5 are readonly.

const config: ReadonlyDeepConfig = {
  level1: { level2: { level3: { level4: { level5: { value: "deep" } } } } },
};

// config.level1.level2.level3.level4.level5.value = "other";  // ❌ Error: readonly.
console.log(config.level1.level2.level3.level4.level5.value);  // "deep"

console.log("Depth-limited DeepReadonly complete.");
```

**Expected Output:**
```
deep
Depth-limited DeepReadonly complete.
```

**Why This Output Occurs:** The depth-limited `DeepReadonlyLimited<T, D>` stops recursion at depth 5, preventing `TS2589` errors while still transforming all levels up to that depth. Beyond depth 5, the type remains unchanged.

### Real-World Cases

**Case 1: Redux State**
Redux state slices use `DeepReadonly<T>` to enforce immutability at all levels, preventing accidental mutations.

**Case 2: Zustand Store**
Zustand stores use `DeepPartial<T>` to type partial state updates, allowing callers to specify only changed fields.

**Case 3: Configuration Management**
Configuration libraries use `DeepPartial<T>` for deep merge operations, enabling partial configuration overrides.

**Case 4: Form Libraries**
Form libraries use `DeepPartial<T>` for partial form updates and `DeepReadonly<T>` for immutable form state.

**Case 5: Immutable Data Structures**
Immer and similar libraries use recursive readonly types to model immutable state after updates.

**Case 6: API Patch Requests**
PATCH endpoints use `DeepPartial<T>` to type partial resource updates at any nesting level.

### References

- ts-deep-types (npm): DeepPartial, DeepRequired, DeepReadonly — https://www.npmjs.com/package/ts-deep-types
- Total TypeScript: Use Recursion and Mapped Types to Create a Type Helper — https://www.totaltypescript.com/use-recursion-and-mapped-types-to-create-a-type-helper
- Stack Overflow: How does this DeepPartial type work from TypeScript? — https://stackoverflow.com/questions/65665777
- FreeCodeCamp: Recursive Types in TypeScript — https://www.freecodecamp.org/news/recursive-types-in-typescript-a-brief-exploration/
- GitHub: Use Tail-Recursion Accumulator Pattern to Bypass the 50-Step Limit — https://raw.githubusercontent.com/pproenca/dot-skills/main/skills/.experimental/typescript-advanced-patterns/references/tlp-tail-recursion-accumulator.md
- utility-types (npm): DeepReadonly, DeepPartial — https://www.npmjs.com/package/utility-types
- GeeksforGeeks: What is Recursive Generic in TypeScript? — https://origin.geeksforgeeks.org/what-is-recursive-generic-in-typescript/


## Summary: Recursive Type Feature Comparison

| Feature | Syntax / Pattern | Purpose | Key Limitation |
|---------|------------------|---------|----------------|
| Recursive type alias | `type Json = string \| Json[] \| { [key: string]: Json }` | Model self-referential data | 50-level compiler limit |
| Recursive interface | `interface ListNode { next?: ListNode }` | Model linked structures | Same limit applies |
| Tree structure | `type TreeNode<T> = { value: T; children: TreeNode<T>[] }` | Model hierarchical data | Depth limited by compiler |
| Linked list | `type ListNode<T> = { value: T; next: ListNode<T> \| null }` | Model sequential data | Recursion depth limited |
| AST modeling | `type Expr = { kind: "binary"; left: Expr; right: Expr }` | Model language syntax | Large unions possible |
| Nested config | `type Config = { [key: string]: ConfigValue }` | Model arbitrary config depth | 50-level limit |
| `DeepReadonly<T>` | `T extends object ? { readonly [K in keyof T]: DeepReadonly<T[K]> } : T` | Make all levels readonly | Unbounded recursion risk |
| `DeepPartial<T>` | `T extends object ? { [K in keyof T]?: DeepPartial<T[K]> } : T` | Make all levels optional | Unbounded recursion risk |
| Depth-limited recursion | `D extends 5 ? T : ... DeepReadonly<T[K], Increment<D>>` | Bound recursion to avoid TS2589 | Adds complexity |
| TS2589 error | `Type instantiation is excessively deep and possibly infinite` | Indicates recursion limit exceeded | Use depth limiter or flatten |
| Compiler limit | ~50 levels per type instantiation | Prevent exponential type-checking | Hard constraint (raised to 500 in TS 4.5) |


## References

- TypeScript 3.7 Release Notes: Recursive Type Aliases — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html
- TypeScript Playground: Recursive Type References — https://www.typescriptlang.org/play/3-7/types-and-code-flow/recursive-type-references.ts.html
- TypeScript Handbook: Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Handbook: Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- GeeksforGeeks: What are Recursive Types & Interfaces in TypeScript? — https://origin.geeksforgeeks.org/what-are-recursive-types-interfaces-in-typescript/
- GeeksforGeeks: What is Recursive Generic in TypeScript? — https://origin.geeksforgeeks.org/what-is-recursive-generic-in-typescript/
- FreeCodeCamp: Recursive Types in TypeScript — https://www.freecodecamp.org/news/recursive-types-in-typescript-a-brief-exploration/
- Stack Overflow: Using recursive type alias in generic results in error — https://stackoverflow.com/questions/60265325
- Stack Overflow: How does this DeepPartial type work from TypeScript? — https://stackoverflow.com/questions/65665777
- GitHub Issue #30188: Recursion limit and type instantiation — https://github.com/microsoft/TypeScript/issues/30188
- GitHub PR #45025: Increase type instantiation depth limit from 50 to 500 — https://github.com/microsoft/TypeScript/pull/45025
- ts-deep-types (npm): DeepPartial, DeepRequired, DeepReadonly — https://www.npmjs.com/package/ts-deep-types
- utility-types (npm): DeepReadonly, DeepPartial — https://www.npmjs.com/package/utility-types
- Total TypeScript: Use Recursion and Mapped Types to Create a Type Helper — https://www.totaltypescript.com/use-recursion-and-mapped-types-to-create-a-type-helper
- TS2589 Usage Guide: Preventing "Type instantiation excessively deep" — https://raw.githubusercontent.com/libar-dev/zod-convex-packages/main/docs/ts2589-usage.md
- GitHub: Use Tail-Recursion Accumulator Pattern to Bypass the 50-Step Limit — https://raw.githubusercontent.com/pproenca/dot-skills/main/skills/.experimental/typescript-advanced-patterns/references/tlp-tail-recursion-accumulator.md