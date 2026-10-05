# TypeScript Arrays: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
TypeScript arrays are ordered collections of values of a specified element type. They extend JavaScript's native array behavior with static type checking, ensuring that all elements conform to a declared type and that array operations (push, pop, map, etc.) are type-safe.

**Technical Definition**
In TypeScript, an array type is a generic type reference to the global `Array<T>` interface, where `T` is the element type. The `Array<T>` interface declares a numeric index signature for the element type, a `length` property, and a comprehensive set of mutating and non-mutating methods (push, pop, map, filter, etc.). Array type literals (`T[]`) are syntactic sugar for `Array<T>` references. TypeScript's type system enforces element type consistency across all array operations, and readonly array types (`ReadonlyArray<T>` or `readonly T[]`) remove all mutating methods from the type interface.

**Beginner-Friendly Explanation**
A TypeScript array is a list of items where every item must be the same type. If you say an array is a `number[]`, TypeScript will make sure you never accidentally put a string into it. This prevents a whole class of bugs where you might mix types and get strange behavior at runtime. Arrays can be written in two ways: `number[]` (the common style) or `Array<number>` (the generic style). Both mean exactly the same thing. TypeScript also has readonly arrays, which are arrays you can read from but never change—like a printed list you can't edit.

### Key Characteristics

- **Homogeneous element types**: All elements in a TypeScript array share the same declared type.
- **Two equivalent syntaxes**: `T[]` and `Array<T>` are functionally identical.
- **Type-safe array methods**: Methods like `push`, `map`, and `filter` are typed based on the element type.
- **Readonly variants**: `ReadonlyArray<T>` and `readonly T[]` prevent mutation at compile time.
- **Inference and widening**: Array literals are inferred as mutable arrays with widened element types by default, unless `as const` is used.
- **Compile-time only**: Array type information is erased during compilation.

### Prerequisites

- Basic knowledge of JavaScript arrays and array methods
- Familiarity with TypeScript primitive types (`string`, `number`, `boolean`)
- Understanding of TypeScript type annotations and inference
- Familiarity with `tsconfig.json` compiler options

### Related Programming Areas

- **Generic Types**: `Array<T>` is a generic interface, and understanding arrays requires basic generic concepts
- **Tuples**: Fixed-length arrays with per-element types, distinct from general arrays
- **Readonly Types**: `ReadonlyArray<T>` is part of TypeScript's readonly modifier system
- **Type Inference**: Array literal inference and widening are part of TypeScript's broader inference system

### Core Concepts / Features

1. Array Type Syntax (`T[]` vs `Array<T>`)
2. Arrays of Primitives and Objects
3. Multidimensional Arrays
4. Readonly Arrays (`ReadonlyArray<T>` and `readonly T[]`)
5. Array Inference and Widening


## 1. Array Type Syntax (`T[]` vs `Array<T>`)

### Definitions

**Core Definition**
TypeScript provides two functionally equivalent syntaxes for declaring array types: the shorthand `T[]` (element type followed by square brackets) and the generic `Array<T>` (the `Array` interface with a type argument). Both syntaxes describe the same type and can be used interchangeably.

**Technical Definition**
The `T[]` syntax is an array type literal, defined in the TypeScript language specification as "an element type followed by an open and close square bracket". This literal is simply shorthand notation for a reference to the generic `Array<T>` interface in the global namespace with the element type as a type argument. Both forms are 100% equivalent in the type system, with the only practical differences being readability, parsing ambiguity with complex types, and behavior with the `keyof` operator.

**Beginner-Friendly Explanation**
TypeScript gives you two ways to write the same thing: `string[]` and `Array<string>`. They both mean "an array of strings." The `string[]` style is shorter and more common—it looks like an array, which makes it intuitive. The `Array<string>` style is more explicit and can be easier to read when the element type is complex (like a union or an object type). You can use either one; TypeScript treats them identically.

### Purposes

- To declare the element type of an array in variable annotations, function parameters, and return types.
- To enable type-safe array operations by propagating the element type to methods like `map`, `filter`, and `reduce`.
- To provide a consistent syntax for array types across the codebase.
- To avoid parsing ambiguity when the element type is a union, intersection, or function type.
- To support `keyof` operations on array element types when using `Array<T>`.

### Syntax Rules and Structure

**General Syntax: Shorthand `T[]`**

```typescript
let variableName: ElementType[] = [element1, element2];
```

**Component Breakdown**
- `ElementType`: The type of each element in the array.
- `[]`: The array type suffix indicating an array of `ElementType`.

**General Syntax: Generic `Array<T>`**

```typescript
let variableName: Array<ElementType> = [element1, element2];
```

**Component Breakdown**
- `Array`: The global generic array interface.
- `<ElementType>`: The type argument specifying the element type.

**General Syntax: Parenthesized `T[]` for Complex Types**

```typescript
let variableName: (Type1 | Type2)[] = [value1, value2];
```

**Component Breakdown**
- `(Type1 | Type2)`: The union element type, enclosed in parentheses.
- `[]`: The array suffix.

**Syntax Rules**

- `T[]` and `Array<T>` are functionally equivalent and can be used interchangeably.
- When the element type is a union, intersection, function, or constructor type, it must be enclosed in parentheses with the `T[]` syntax: `(string | number)[]`.
- The `Array<T>` syntax does not require parentheses for complex element types: `Array<string | number>`.
- `T[][]` represents a two-dimensional array (array of arrays).
- `Array<Array<T>>` is equivalent to `T[][]`.
- The `readonly` modifier can be combined with either syntax: `readonly T[]` or `ReadonlyArray<T>`.

**Constraints and Limitations**

- Using `keyof` with `T[]` can lead to unexpected results because `[]` resolves before `keyof`: `keyof Person[]` means `keyof (Person[])`, not `(keyof Person)[]`.
- The fix is to either use parentheses `(keyof Person)[]` or use `Array<keyof Person>`.
- TypeScript's editor tooltips and error messages always display the `T[]` syntax, regardless of which syntax the developer wrote.
- The `typescript-eslint` `array-type` rule can enforce consistency, either always using `T[]` or always using `Array<T>`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `T[]` and `Array<T>` Equivalence

```typescript
// Step 1: Declare an array using T[] syntax.
let numbers1: number[] = [1, 2, 3];

// Step 2: Declare an array using Array<T> syntax.
let numbers2: Array<number> = [4, 5, 6];

// Step 3: Both variables have the same type and behavior.
numbers1.push(4);  // ✅ Allowed
numbers2.push(7);  // ✅ Allowed

// Step 4: Assign one to the other — they are fully compatible.
numbers1 = numbers2;  // ✅ No error — identical types

console.log(numbers1);  // [4, 5, 6, 7]
```

**Expected Output:**
```
[4, 5, 6, 7]
```

**Why This Output Occurs:** `number[]` and `Array<number>` are the same type in TypeScript's type system. The assignment `numbers1 = numbers2` is allowed because the types are structurally and nominally identical. Both variables share the same element type and method signatures.

#### Example 2: Parentheses for Complex Element Types

```typescript
// Step 1: Union element type requires parentheses with T[].
let mixed1: (string | number)[] = ["hello", 42, "world", 100];

// Step 2: Array<T> does not require parentheses.
let mixed2: Array<string | number> = ["foo", 1, "bar", 2];

// Step 3: Without parentheses, TypeScript parses this incorrectly.
// let invalid: string | number[] = ["hello", 42];
// ❌ Error: Type 'string' is not assignable to type 'string | number[]'.

// Step 4: Function type element requires parentheses.
let callbacks: (() => void)[] = [
  () => console.log("First"),
  () => console.log("Second"),
];

// Step 5: Call each callback.
callbacks.forEach((cb) => cb());
```

**Expected Output:**
```
First
Second
```

**Why This Output Occurs:** The `(string | number)[]` syntax requires parentheses because without them, TypeScript would parse `string | number[]` as "a string OR an array of numbers" rather than "an array of strings or numbers." The `Array<string | number>` syntax avoids this ambiguity entirely. Function types similarly require parentheses in the `T[]` syntax.

### Real-World Cases

**Case 1: API Response Arrays**
When typing API responses that return arrays of objects, using `Array<User>` makes the element type explicit and readable, especially when the `User` interface is complex. Teams that prefer explicitness often standardize on `Array<T>`.

**Case 2: Function Signatures**
Function parameters that accept arrays benefit from `T[]` for simple types (`string[]`) and `Array<T>` for complex types (`Array<keyof Config>`). This balances brevity with clarity.

**Case 3: ESLint Enforcement**
Large codebases often use the `@typescript-eslint/array-type` rule to enforce one syntax consistently, preventing stylistic drift. The rule can be configured to prefer `T[]` for simple types and `Array<T>` for complex types.

---

## 2. Arrays of Primitives and Objects

### Definitions

**Core Definition**
TypeScript arrays can hold elements of any type: primitive values (string, number, boolean) or object types (interfaces, classes, object literals). The element type determines what values are allowed in the array and what operations are type-safe.

**Technical Definition**
An array of primitives is declared as `T[]` or `Array<T>` where `T` is a primitive type (`string`, `number`, `boolean`, etc.). An array of objects uses an interface, class, or anonymous object type as the element type. TypeScript enforces that every element in the array is assignable to the declared element type, and array methods return values typed according to the element type. For object arrays, TypeScript's structural typing allows objects that have at least the required properties to be included.

**Beginner-Friendly Explanation**
You can make an array of anything: numbers, strings, or even complex objects. If you have an array of `User` objects, TypeScript knows that every item in the array has `name` and `email` properties, so you can safely access those properties without checking. If you try to put a string into a `User[]`, TypeScript stops you. This works because TypeScript checks every element against the declared type, whether the array is created with a literal or populated later with `push`.

### Purposes

- To store and manipulate collections of primitive values with type safety.
- To manage collections of domain objects (users, orders, products) with compile-time shape checking.
- To enable type-safe iteration and transformation of object collections using array methods.
- To enforce that all elements in a collection conform to a consistent structure.
- To support functional programming patterns (map, filter, reduce) with typed object collections.

### Syntax Rules and Structure

**General Syntax: Array of Primitives**

```typescript
let variableName: PrimitiveType[] = [value1, value2, value3];
```

**Component Breakdown**
- `PrimitiveType`: `string`, `number`, `boolean`, etc.
- The array can only contain values of the primitive type.

**General Syntax: Array of Objects**

```typescript
interface ObjectType {
  property1: Type1;
  property2: Type2;
}

let variableName: ObjectType[] = [
  { property1: value1, property2: value2 },
  { property1: value3, property2: value4 },
];
```

**Component Breakdown**
- `ObjectType`: An interface, type alias, or class name.
- Each object literal in the array must conform to `ObjectType`.

**Syntax Rules**

- Array element types can be any valid TypeScript type: primitives, objects, unions, intersections, generics.
- Object arrays use structural typing: objects must have at least the required properties, but may have additional properties (when not using object literals directly).
- Array methods like `map`, `filter`, and `find` return values typed by the element type.
- `find` returns `ElementType | undefined` because the element may not be found.
- `filter` returns `ElementType[]` with the same element type.

**Constraints and Limitations**

- Direct object literals with extra properties trigger excess property checking.
- Arrays of union types require narrowing before accessing type-specific properties.
- Empty arrays infer the element type as `never[]` unless annotated or contextualized.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Array of Primitives

```typescript
// Step 1: Declare an array of strings.
let names: string[] = ["Alice", "Bob", "Charlie"];

// Step 2: Use type-safe methods.
names.push("Dave");  // ✅ Allowed
// names.push(42);   // ❌ Error: Argument of type 'number' is not assignable to 'string'.

// Step 3: Transform the array with map.
let upperNames = names.map((name) => name.toUpperCase());
console.log(upperNames);  // ["ALICE", "BOB", "CHARLIE", "DAVE"]

// Step 4: Filter the array.
let shortNames = names.filter((name) => name.length <= 4);
console.log(shortNames);  // ["Bob", "Dave"]
```

**Expected Output:**
```
[ 'ALICE', 'BOB', 'CHARLIE', 'DAVE' ]
[ 'Bob', 'Dave' ]
```

**Why This Output Occurs:** The `string[]` type ensures that `push(42)` is a compile error because `42` is not a string. The `map` method infers the callback parameter as `string` and returns a `string[]`. The `filter` method returns a `string[]` containing only elements that satisfy the predicate.

#### Example 2: Array of Objects

```typescript
// Step 1: Define an interface for the object type.
interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

// Step 2: Create an array of Product objects.
const products: Product[] = [
  { id: 1, name: "Laptop", price: 999, inStock: true },
  { id: 2, name: "Mouse", price: 29, inStock: true },
  { id: 3, name: "Keyboard", price: 79, inStock: false },
];

// Step 3: Access object properties with type safety.
const availableProducts = products.filter((p) => p.inStock);
console.log(availableProducts.map((p) => `${p.name}: $${p.price}`));

// Step 4: Find a specific product.
const laptop = products.find((p) => p.id === 1);
// laptop is Product | undefined
if (laptop) {
  console.log(`Found: ${laptop.name}`);
}
```

**Expected Output:**
```
[ 'Laptop: $999', 'Mouse: $29' ]
Found: Laptop
```

**Why This Output Occurs:** The `Product[]` type ensures that every element has the required properties. The `filter` callback receives a `Product` parameter, so `p.inStock` is type-safe. The `find` method returns `Product | undefined`, requiring the null check before accessing `laptop.name`.

### Real-World Cases

**Case 1: E-Commerce Product Catalogs**
Online stores manage arrays of product objects with properties like `id`, `name`, `price`, and `category`. TypeScript ensures that filtering, sorting, and mapping operations are type-safe, preventing runtime errors when accessing product properties.

**Case 2: User Management Systems**
Arrays of user objects with `id`, `email`, `role`, and `permissions` benefit from type safety when checking permissions or filtering users by role.

**Case 3: Data Visualization**
Charting libraries receive arrays of data point objects (e.g., `{ x: number; y: number }`). TypeScript ensures that data transformations (scaling, filtering) maintain the correct element shape.

---

## 3. Multidimensional Arrays

### Definitions

**Core Definition**
A multidimensional array in TypeScript is an array whose elements are themselves arrays. The most common form is the two-dimensional array (matrix), declared as `T[][]` or `Array<Array<T>>`. Each level of nesting adds a dimension, and TypeScript enforces the element type at every level.

**Technical Definition**
A multidimensional array is represented in TypeScript as a nested generic array type: `T[][]` is equivalent to `Array<Array<T>>` and describes an array where every element is an array of `T`. For an N-dimensional array, the type has N levels of array nesting: `T[][][]` for three dimensions. TypeScript does not enforce uniform lengths across inner arrays—a `number[][]` can contain arrays of different lengths, making it a jagged array rather than a strict matrix.

**Beginner-Friendly Explanation**
A multidimensional array is an array of arrays. Imagine a spreadsheet: you have rows, and each row has cells. In TypeScript, a two-dimensional array of numbers is written as `number[][]`. The first `[]` means "array of" and the second `[]` means "array of numbers." So `number[][]` means "an array of arrays of numbers." You can go deeper: `number[][][]` is a three-dimensional array. TypeScript checks that every element at every level has the right type.

### Purposes

- To represent matrices and grids for mathematical computations.
- To model tabular data with rows and columns.
- To store game boards, seating charts, and other 2D layouts.
- To handle image data as 2D or 3D pixel arrays.
- To provide type-safe access to nested array structures.

### Syntax Rules and Structure

**General Syntax: 2D Array**

```typescript
let variableName: ElementType[][] = [
  [element1, element2],
  [element3, element4],
];
```

**Component Breakdown**
- `ElementType[][]`: The element type followed by two array suffixes.
- The outer `[]` indicates an array of rows.
- The inner `[]` indicates an array of `ElementType` values.

**General Syntax: Generic 2D Array**

```typescript
let variableName: Array<Array<ElementType>> = [
  [element1, element2],
  [element3, element4],
];
```

**Component Breakdown**
- `Array<Array<ElementType>>`: Nested generic array types.
- Equivalent to `ElementType[][]`.

**General Syntax: 3D Array**

```typescript
let variableName: ElementType[][][] = [
  [[element1], [element2]],
  [[element3], [element4]],
];
```

**Component Breakdown**
- Three levels of array nesting for three dimensions.

**Syntax Rules**

- Each additional dimension adds one `[]` suffix (or one `Array<>` nesting level).
- `T[][]` and `Array<Array<T>>` are equivalent.
- Inner arrays may have different lengths (jagged arrays).
- Accessing an element requires one index per dimension: `matrix[row][col]`.
- TypeScript does not enforce rectangular (uniform) dimensions by default.

**Constraints and Limitations**

- TypeScript cannot enforce that all inner arrays have the same length without tuple types.
- Initializing multidimensional arrays requires nested loops or explicit literals.
- Deeply nested arrays (4+ dimensions) become difficult to read and maintain; consider custom types or libraries.
- `Array<Array<T>>` can be more readable than `T[][]` for complex element types.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Two-Dimensional Array (Matrix)

```typescript
// Step 1: Declare a 2D array of numbers (3x3 matrix).
const matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];

// Step 2: Access elements by row and column.
console.log(matrix[0][0]);  // 1 (first row, first column)
console.log(matrix[1][2]);  // 6 (second row, third column)

// Step 3: Iterate over the matrix.
for (let row = 0; row < matrix.length; row++) {
  for (let col = 0; col < matrix[row].length; col++) {
    process.stdout.write(`${matrix[row][col]} `);
  }
  console.log();
}

// Step 4: Calculate the sum of all elements.
const sum = matrix.flat().reduce((acc, val) => acc + val, 0);
console.log(`Sum: ${sum}`);
```

**Expected Output:**
```
1
6
1 2 3 
4 5 6 
7 8 9 
Sum: 45
```

**Why This Output Occurs:** The `number[][]` type declares a 2D array of numbers. `matrix[0][0]` accesses the first row's first element. The nested loops iterate over rows and columns. The `flat()` method flattens the 2D array into a 1D array, enabling `reduce` to sum all elements.

#### Example 2: Jagged Array and Type Safety

```typescript
// Step 1: Declare a jagged array (rows of different lengths).
const jagged: string[][] = [
  ["a", "b"],
  ["c"],
  ["d", "e", "f"],
];

// Step 2: Access elements — inner arrays may have different lengths.
console.log(jagged[0].length);  // 2
console.log(jagged[1].length);  // 1
console.log(jagged[2].length);  // 3

// Step 3: TypeScript enforces the element type at every level.
// jagged[0].push(42);  // ❌ Error: number is not assignable to string

// Step 4: Process each row.
jagged.forEach((row, index) => {
  console.log(`Row ${index}: ${row.join(", ")}`);
});
```

**Expected Output:**
```
2
1
3
Row 0: a, b
Row 1: c
Row 2: d, e, f
```

**Why This Output Occurs:** The `string[][]` type enforces that each inner array contains strings. The inner arrays can have different lengths because TypeScript does not require rectangular dimensions. Accessing `jagged[0].length` returns the length of the first inner array.

### Real-World Cases

**Case 1: Game Boards**
Chess, checkers, and tic-tac-toe boards are naturally represented as 2D arrays. A chess board is `Piece | null[][]` (8x8), where each cell contains a piece or is empty.

**Case 2: Image Processing**
Images are often represented as 2D arrays of pixels. A grayscale image is `number[][]` (brightness values), and a color image is `[number, number, number][][]` (RGB triplets per pixel).

**Case 3: Spreadsheet Data**
Tabular data with rows and columns can be modeled as `CellValue[][]`, where each row is an array of cell values. TypeScript ensures that all cells conform to the expected value type.

---

## 4. Readonly Arrays (`ReadonlyArray<T>` and `readonly T[]`)

### Definitions

**Core Definition**
A readonly array in TypeScript is an array that cannot be modified after creation. TypeScript provides two syntaxes for readonly arrays: `ReadonlyArray<T>` and `readonly T[]`. Both remove all mutating methods (`push`, `pop`, `splice`, etc.) from the array type and prevent assignment to array indices. Readonly arrays can still be read from and transformed using non-mutating methods like `map`, `filter`, `slice`, and `concat`.

**Technical Definition**
`ReadonlyArray<T>` is a TypeScript interface that is structurally identical to `Array<T>` but with all mutating methods removed. The `readonly T[]` syntax is syntactic sugar for `ReadonlyArray<T>`, just as `T[]` is for `Array<T>`. A `ReadonlyArray<T>` is not assignable to `Array<T>` because the latter requires mutating methods that the former lacks. However, `Array<T>` is assignable to `ReadonlyArray<T>` because the mutable array has all the readonly array's members (and more). The `as const` assertion on an array literal produces a readonly tuple type, which is a more specific form of readonly array.

**Beginner-Friendly Explanation**
A readonly array is an array you can look at but not change. You can read elements, loop through them, and create new arrays from them—but you can't add, remove, or replace elements. TypeScript gives you two ways to write it: `ReadonlyArray<number>` or `readonly number[]`. Both mean the same thing. The key benefit is that if you pass a readonly array to a function, the function can't accidentally modify your data. Readonly arrays are especially useful for configuration lists, constant data, and function parameters that should not mutate their inputs.

### Purposes

- To prevent accidental mutation of arrays that should remain constant.
- To document the intended immutability of array data in function signatures and interfaces.
- To enable safer functional programming by discouraging in-place mutation.
- To allow mutable arrays to be passed to functions that only read from arrays.
- To work with `as const` assertions to create deeply immutable data structures.

### Syntax Rules and Structure

**General Syntax: `ReadonlyArray<T>`**

```typescript
let variableName: ReadonlyArray<ElementType> = [element1, element2];
```

**Component Breakdown**
- `ReadonlyArray`: The readonly array interface.
- `<ElementType>`: The element type.

**General Syntax: `readonly T[]`**

```typescript
let variableName: readonly ElementType[] = [element1, element2];
```

**Component Breakdown**
- `readonly`: The modifier keyword.
- `ElementType[]`: The array type with the readonly modifier applied.

**General Syntax: Readonly Array in Function Parameters**

```typescript
function functionName(parameter: readonly ElementType[]): ReturnType {
  // parameter cannot be mutated
}
```

**Component Breakdown**
- `readonly ElementType[]`: The readonly array type.
- The function cannot call mutating methods on `parameter`.

**Syntax Rules**

- `ReadonlyArray<T>` and `readonly T[]` are equivalent syntaxes.
- Mutating methods (`push`, `pop`, `splice`, `shift`, `unshift`, `sort`, `reverse`, `fill`, `copyWithin`) are not available on readonly arrays.
- Non-mutating methods (`map`, `filter`, `slice`, `concat`, `forEach`, `find`, `some`, `every`, `reduce`) are available.
- Index assignment (`array[0] = value`) is a compile error on readonly arrays.
- `Array<T>` is assignable to `ReadonlyArray<T>`, but `ReadonlyArray<T>` is not assignable to `Array<T>`.
- `as const` on an array literal produces a `readonly` tuple type (e.g., `readonly [1, 2, 3]`).

**Constraints and Limitations**

- Readonly arrays are shallow: the array itself cannot be modified, but the objects it contains may still be mutable.
- `readonly` is erased at runtime and provides no runtime protection.
- Some third-party libraries expect mutable arrays and may require type assertions or copies.
- The `readonly` modifier does not work with `Array<T>` syntax directly; use `ReadonlyArray<T>` instead.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Readonly Array Basics

```typescript
// Step 1: Declare a readonly array using ReadonlyArray<T>.
const environments: ReadonlyArray<string> = ["dev", "staging", "prod"];

// Step 2: Declare a readonly array using readonly T[].
const ports: readonly number[] = [3000, 3001, 3002];

// Step 3: Reading is allowed.
console.log(environments[0]);  // "dev"
console.log(ports.length);     // 3

// Step 4: Mutating methods are NOT available.
// environments.push("test");  // ❌ Error: Property 'push' does not exist on type 'readonly string[]'.
// ports[0] = 8080;            // ❌ Error: Index signature in type 'readonly number[]' only permits reading.

// Step 5: Non-mutating methods are available.
const upperEnvs = environments.map((e) => e.toUpperCase());
console.log(upperEnvs);  // ["DEV", "STAGING", "PROD"]

// Step 6: Mutable arrays are assignable to readonly arrays.
const mutablePorts: number[] = [4000, 4001];
const readonlyPorts: readonly number[] = mutablePorts;  // ✅ Allowed
// But readonly arrays are NOT assignable to mutable arrays.
// mutablePorts = readonlyPorts;  // ❌ Error
```

**Expected Output:**
```
dev
3
[ 'DEV', 'STAGING', 'PROD' ]
```

**Why This Output Occurs:** The `ReadonlyArray<string>` and `readonly number[]` types remove all mutating methods. The `push` and index assignment operations are compile errors. Non-mutating methods like `map` work because they return new arrays without modifying the original. The assignment from `number[]` to `readonly number[]` is allowed because the mutable array satisfies all requirements of the readonly array.

#### Example 2: Readonly Arrays in Function Parameters

```typescript
// Step 1: Define a function that accepts a readonly array.
function calculateTotal(prices: readonly number[]): number {
  // prices.push(100);  // ❌ Error: Cannot mutate a readonly array
  return prices.reduce((sum, price) => sum + price, 0);
}

// Step 2: Pass a mutable array — this is allowed.
const mutablePrices = [10, 20, 30];
console.log(calculateTotal(mutablePrices));  // 60

// Step 3: Pass a readonly array directly.
const readonlyPrices: readonly number[] = [5, 15, 25];
console.log(calculateTotal(readonlyPrices));  // 45

// Step 4: The function cannot mutate the array.
// calculateTotal(mutablePrices);  // mutablePrices remains [10, 20, 30]
console.log(mutablePrices);  // [10, 20, 30]
```

**Expected Output:**
```
60
45
[ 10, 20, 30 ]
```

**Why This Output Occurs:** The `readonly number[]` parameter type prevents the function from calling `push` or any other mutating method. The function can still use `reduce` because it is a non-mutating method. The mutable array `mutablePrices` is unchanged after the function call, demonstrating that the readonly parameter type prevents mutation.

### Real-World Cases

**Case 1: Configuration Constants**
Application configuration arrays (e.g., supported locales, allowed HTTP methods) should be readonly to prevent accidental modification at runtime. Using `ReadonlyArray<string>` or `readonly string[]` documents and enforces this intent.

**Case 2: Function Parameter Protection**
Functions that only read from arrays should accept `readonly T[]` parameters to prevent accidental mutation of caller-provided data. This is a common pattern in functional programming and library design.

**Case 3: `as const` for Literal Preservation**
Using `as const` on an array literal creates a readonly tuple with preserved literal types. This is useful for defining constant lists of string literals that need to be used as types.

---

## 5. Array Inference and Widening

### Definitions

**Core Definition**
Array inference is TypeScript's ability to determine the element type of an array literal without an explicit annotation. Array widening is the process by which TypeScript broadens literal element types (e.g., `"GET"`) to their general types (e.g., `string`) in mutable array contexts. The `as const` assertion can prevent widening and preserve literal types.

**Technical Definition**
When TypeScript infers the type of an array literal, it computes the best common type of all elements. In a mutable context (e.g., `let` declaration or a mutable array), literal element types are widened to their base types: `["GET", "POST"]` is inferred as `string[]`, not `("GET" | "POST")[]`. The widening rules are defined in the TypeScript specification: "The type inferred for an element in an array literal is the widened literal type of the expression unless the element has a contextual type that includes literal types". The `as const` assertion on an array literal prevents widening and produces a readonly tuple type with preserved literal types. Since TypeScript 3.4, `as const` is the recommended way to achieve narrow array types without verbose annotations.

**Beginner-Friendly Explanation**
When you write `let methods = ["GET", "POST"]`, TypeScript thinks: "This is a mutable array, so the elements could be changed to any string. I'll type it as `string[]`." This is called widening—TypeScript widens the specific strings `"GET"` and `"POST"` to the general type `string`. But sometimes you want TypeScript to remember the exact values. You can use `as const` to tell TypeScript: "Don't widen these—keep them as `"GET"` and `"POST"`." This creates a readonly tuple with the exact literal types. Widening is helpful for mutable arrays, but `as const` is essential when you need the specific values as types.

### Purposes

- To automatically infer the element type of array literals without explicit annotations.
- To widen literal types to general types in mutable array contexts for flexibility.
- To prevent widening and preserve literal types using `as const` for precise typing.
- To enable type-safe discriminated unions and template literal types from array literals.
- To control the trade-off between type precision and mutability.

### Syntax Rules and Structure

**General Syntax: Widened Inference (Default)**

```typescript
let variableName = [element1, element2, element3];
// Inferred as: ElementBaseType[]
```

**Component Breakdown**
- `[element1, element2, element3]`: Array literal with literal elements.
- The inferred type is a mutable array of the widened element type.

**General Syntax: Non-Widened Inference with `as const`**

```typescript
const variableName = [element1, element2, element3] as const;
// Inferred as: readonly [LiteralType1, LiteralType2, LiteralType3]
```

**Component Breakdown**
- `as const`: The const assertion that prevents widening.
- The inferred type is a readonly tuple with preserved literal types.

**General Syntax: Tuple Hint**

```typescript
const variableName: [Type1, Type2] = [element1, element2];
// Inferred as: [Type1, Type2]
```

**Component Breakdown**
- `[Type1, Type2]`: A tuple type annotation provides a context that hints tuple inference.
- The literal is inferred as a tuple rather than a general array.

**Syntax Rules**

- Array literal element types are widened to their base types in mutable contexts.
- `const` assertions (`as const`) prevent widening and create readonly tuples.
- Tuple type annotations provide a context that hints tuple inference.
- The `satisfies` operator (TypeScript 4.9+) can validate array literals against a type without widening.
- Contextual types can influence array inference: if the expected type is a tuple, the literal is inferred as a tuple.

**Constraints and Limitations**

- `as const` on an array literal creates a readonly tuple, which may be more specific than desired (fixed length).
- Widening is not always predictable with complex expressions.
- The `as const` assertion cannot be combined with an explicit type annotation in the same declaration.
- Empty arrays without annotations or context infer as `never[]` (or `any[]` in evolving array scenarios).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Default Widening vs `as const`

```typescript
// Step 1: Default inference — widening occurs.
let methods = ["GET", "POST", "PUT"];
// Inferred type: string[]
// The literal types "GET", "POST", "PUT" are widened to string.

// Step 2: The array is mutable, so push accepts any string.
methods.push("DELETE");  // ✅ Allowed

// Step 3: as const prevents widening.
const methodsConst = ["GET", "POST", "PUT"] as const;
// Inferred type: readonly ["GET", "POST", "PUT"]
// Literal types are preserved, and the array is readonly.

// Step 4: Mutation is prevented.
// methodsConst.push("DELETE");  // ❌ Error: Property 'push' does not exist.

// Step 5: Extract the union of literal types.
type HttpMethod = (typeof methodsConst)[number];
// HttpMethod = "GET" | "POST" | "PUT"

// Step 6: Use the union in a function signature.
function makeRequest(method: HttpMethod): void {
  console.log(`Request method: ${method}`);
}
makeRequest("GET");  // ✅ Allowed
// makeRequest("PATCH");  // ❌ Error: Argument of type '"PATCH"' is not assignable to parameter of type 'HttpMethod'.
```

**Expected Output:**
```
Request method: GET
```

**Why This Output Occurs:** The `let methods` array is widened to `string[]` because it is mutable. The `methodsConst` array uses `as const`, so it is inferred as `readonly ["GET", "POST", "PUT"]` with preserved literal types. The indexed access type `(typeof methodsConst)[number]` extracts the union of literal types, enabling type-safe function parameters.

#### Example 2: Tuple Inference with Contextual Types

```typescript
// Step 1: Without context, an array literal is inferred as a general array.
const point1 = [10, 20];
// Inferred type: number[]

// Step 2: With a tuple type annotation, it is inferred as a tuple.
const point2: [number, number] = [10, 20];
// Inferred type: [number, number]

// Step 3: With as const, it is inferred as a readonly tuple.
const point3 = [10, 20] as const;
// Inferred type: readonly [10, 20]

// Step 4: Tuple types preserve fixed length and per-element types.
function distance([x, y]: [number, number]): number {
  return Math.sqrt(x ** 2 + y ** 2);
}

console.log(distance([3, 4]));    // 5
console.log(distance([5, 12]));   // 13
```

**Expected Output:**
```
5
13
```

**Why This Output Occurs:** The `point1` array is inferred as `number[]` because there is no contextual type. The `point2` tuple annotation provides a contextual type, so the literal is inferred as a tuple. The `point3` array uses `as const`, producing a readonly tuple with preserved literal types. The `distance` function accepts a tuple, enabling destructuring with typed elements.

### Real-World Cases

**Case 1: HTTP Method Constants**
Defining HTTP methods as `const methods = ["GET", "POST", "PUT"] as const` preserves the literal types, enabling type-safe function parameters and discriminated unions for request handling.

**Case 2: Configuration Lists**
Configuration arrays that should not be mutated use `as const` to create readonly tuples, preventing accidental modification and enabling `typeof` extraction for type-level programming.

**Case 3: Redux Action Types**
Redux action type constants benefit from `as const` arrays, which preserve literal types for use in discriminated union narrowing.

---

## References

- TypeScript Handbook: Everyday Types (Arrays) — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#arrays
- TypeScript Language Specification: Array Type Literals — https://chromium.googlesource.com/external/github.com/kythe/kythe/+/7c2c91248ad9bc8d22dfaa00764c6381af512113/third_party/typescript/doc/spec.md#3-8-4-array-type-literals
- Total TypeScript: Array<T> vs T[] — https://www.totaltypescript.com/array-types-in-typescript
- TypeScript ESLint: array-type Rule — https://typescript-eslint.io/rules/array-type/
- TypeScript Handbook: Object Types (Readonly Arrays) — https://www.typescriptlang.org/docs/handbook/2/objects.html
- Egghead: Prevent Type Widening of Array Literals with const Assertions — https://egghead.io/lessons/typescript-prevent-type-widening-of-array-literals-with-typescript-s-const-assertions
- TypeScript 3.4 Release Notes (Const Assertions) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html
- Effective TypeScript: Item 41 — Understand Evolving any — https://effectivetypescript.com/2020/03/09/item41-evolving-any/
- TypeScript Playground: Arrays and Tuples — https://www.typescriptlang.org/play/typescript/arrays-and-tuples.ts.html
- MDN: Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array