# JavaScript Array Destructuring — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Array destructuring is a JavaScript expression that unpacks values from iterables (typically arrays) into distinct variables using a syntax that mirrors array literals.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §13.15.5), array destructuring uses `ArrayAssignmentPattern` and `ArrayBindingPattern` grammar productions. The pattern `[a, b]` iterates the source using the iterator protocol, extracting values by position and binding them to variables. Patterns support default values (`= default`), elisions (skipping via `,`), rest elements (`...rest`), nested patterns, and swapping. Destructuring works with `let`, `const`, `var`, or as a plain assignment expression.

**Beginner-Friendly Explanation:** Array destructuring lets you pull values out of an array and put them into variables in one line. Instead of writing `const first = arr[0]; const second = arr[1];`, you write `const [first, second] = arr;`. It's like unpacking a suitcase — you take out exactly what you need by position.

### Key Characteristics

- **Position-based:** Values are matched by index, not by name.
- **Works on any iterable:** Arrays, strings, Sets, Maps, generators, NodeLists.
- **Supports defaults:** Fallback values when elements are `undefined`.
- **Supports skipping:** Use commas to skip positions.
- **Supports rest:** Collect remaining elements into a new array.
- **Supports nesting:** Destructure deeply nested arrays.
- **Enables swapping:** Exchange variable values without a temporary.

### Prerequisites

- Basic JavaScript syntax (variables, arrays, values).
- Understanding of array indexing and `length`.
- Familiarity with `let`, `const`, and `var`.
- Awareness of iterables and the iterator protocol.
- Basic knowledge of function parameters.

### Related Programming Areas

- **Function parameters:** Destructuring in function signatures.
- **React hooks:** `const [state, setState] = useState(0)`.
- **Iteration:** `for (const [k, v] of map)`.
- **Object destructuring:** Complementary syntax for objects.
- **Spread/rest:** Related `...` syntax.
- **Module imports:** `import` uses similar syntax.

---

## Array Destructuring

### Definitions

**Core Definition:** Array destructuring extracts values from an array into individual variables by matching positions.

**Technical Definition:** Per ECMA-262 §13.15.5, an `ArrayAssignmentPattern` consists of zero or more `AssignmentElement` productions enclosed in `[ ]`. During evaluation, `DestructuringAssignmentEvaluation` obtains the iterator from the source via `GetIterator`, calls `next()` for each element, and assigns the `value` to the target. Holes (elisions) skip positions.

**Beginner-Friendly Explanation:** Array destructuring is a shortcut for pulling values out of an array by their position.

### Purposes

- To extract multiple values from an array concisely.
- To avoid repetitive index access.
- To provide default values for missing elements.
- To swap variables without a temporary.
- To destructure function return values.
- To simplify iteration and parameter handling.

### Syntax Rules and Structure

**General Syntax:**
```js
const [a, b, c] = array;
let [x, y] = array;
var [p, q] = array;
[a, b] = array; // Assignment (no declaration)
```

**Component Breakdown:**
- `[ ]`: The destructuring pattern.
- `a, b, c`: Variable names to bind.
- `array`: The source iterable.
- Each variable receives the value at its index.

**Syntax Rules:**
- Works on any iterable, not just arrays.
- Missing elements are assigned `undefined`.
- Works with `let`, `const`, `var`, or as an assignment.
- In assignment context, no parentheses are needed (unlike object destructuring).
- Default values apply only when the value is `undefined`.
- Rest elements must be last.
- Cannot destructure `null` or `undefined` (throws `TypeError`).

**Constraints and Limitations:**
- Position-based; no named matching.
- Cannot destructure non-iterable objects.
- Destructuring `null` or `undefined` throws `TypeError`.
- Defaults do not trigger on `null`.

### Annotated Code Examples

#### Example 1: Basic Array Destructuring

```js
// Step 1: Define a source array.
const colors = ["red", "green", "blue"];

// Step 2: Destructure into variables.
const [first, second, third] = colors;

// Step 3: Use the variables.
console.log(first);  // "red"
console.log(second); // "green"
console.log(third);  // "blue"
```

**Expected Output:**
```
red
green
blue
```

**Why This Output:** The pattern `[first, second, third]` binds each variable to the value at the corresponding index.

#### Example 2: Fewer Variables Than Elements

```js
const [a, b] = [1, 2, 3, 4];
console.log(a); // 1
console.log(b); // 2
// Elements 3 and 4 are ignored.
```

**Expected Output:**
```
1
2
```

**Why This Output:** Destructuring extracts only as many values as there are variables in the pattern.

#### Example 3: More Variables Than Elements

```js
const [x, y, z] = [1, 2];
console.log(x); // 1
console.log(y); // 2
console.log(z); // undefined
```

**Expected Output:**
```
1
2
undefined
```

**Why This Output:** Missing elements are assigned `undefined`.

#### Example 4: Destructuring from Function Return

```js
function getCoordinates() {
  return [10, 20];
}

const [x, y] = getCoordinates();
console.log(x, y); // 10 20
```

**Expected Output:**
```
10 20
```

**Why This Output:** The function returns an array, which is immediately destructured.

#### Example 5: Works on Any Iterable

```js
const [a, b, c] = "abc";
console.log(a, b, c); // "a" "b" "c"

const [first] = new Set([1, 2, 3]);
console.log(first); // 1

const [k, v] = new Map([["key", "value"]]);
console.log(k, v); // "key" "value"
```

**Expected Output:**
```
a b c
1
key value
```

**Why This Output:** Destructuring uses the iterator protocol, so it works on any iterable.

### Real-World Cases

- **React hooks:** `const [count, setCount] = useState(0);`.
- **API responses:** `const [data, error] = await fetchData();`.
- **Swap operations:** `[a, b] = [b, a];`.
- **Coordinate pairs:** `const [x, y] = point;`.

---

## Default Values

### Definitions

**Core Definition:** Default values in array destructuring provide a fallback when an element is `undefined` or missing.

**Technical Definition:** Per ECMA-262 §13.15.5, an `AssignmentElement` can include `= AssignmentExpression`. During evaluation, if the extracted value is `undefined`, the default expression is evaluated and used.

**Beginner-Friendly Explanation:** Default values let you say "if this position is empty, use this value instead."

### Purposes

- To provide fallbacks for missing elements.
- To make destructuring robust against short arrays.
- To define defaults for optional parameters.

### Syntax Rules and Structure

```js
const [a = 1, b = 2, c = 3] = array;
```

**Component Breakdown:**
- `a`, `b`, `c`: Variable names.
- `= 1`, `= 2`, `= 3`: Default expressions.

**Syntax Rules:**
- Defaults apply only when the value is `undefined`.
- `null` does NOT trigger the default.
- Default expressions can be any expression.
- Can reference earlier bindings in the same pattern.
- Evaluated lazily (only when needed).

**Constraints and Limitations:**
- `null` does not trigger defaults (use `??` or explicit checks).
- Default expressions cannot reference later bindings.
- Side effects in default expressions are called only when needed.

### Annotated Code Examples

#### Example 1: Basic Defaults

```js
const [a = 1, b = 2, c = 3] = [10, undefined];
console.log(a); // 10 (present)
console.log(b); // 2 (default, because undefined)
console.log(c); // 3 (default, because missing)
```

**Expected Output:**
```
10
2
3
```

**Why This Output:** `a` is present; `b` is `undefined`; `c` is missing. Defaults apply to `b` and `c`.

#### Example 2: `null` Does Not Trigger Defaults

```js
const [a = 1, b = 2] = [null, undefined];
console.log(a); // null (not 1!)
console.log(b); // 2
```

**Expected Output:**
```
null
2
```

**Why This Output:** Defaults only trigger on `undefined`. `null` is a valid value.

#### Example 3: Defaults with Expressions

```js
const [a = 1, b = a * 2, c = a + b] = [];
console.log(a); // 1
console.log(b); // 2
console.log(c); // 3
```

**Expected Output:**
```
1
2
3
```

**Why This Output:** Default expressions can reference earlier bindings in the same pattern.

#### Example 4: Function Parameters with Defaults

```js
function greet([name = "Anonymous", greeting = "Hello"] = []) {
  return `${greeting}, ${name}!`;
}

console.log(greet(["Alice"]));         // "Hello, Alice!"
console.log(greet(["Bob", "Hi"]));      // "Hi, Bob!"
console.log(greet());                   // "Hello, Anonymous!"
```

**Expected Output:**
```
Hello, Alice!
Hi, Bob!
Hello, Anonymous!
```

**Why This Output:** The parameter is destructured with defaults; the outer `= []` allows calling without arguments.

### Real-World Cases

- **Configuration:** `const [host = "localhost", port = 3000] = config;`.
- **Function parameters:** Optional positional arguments.
- **State pairs:** `const [state = initialState, setState] = useState();`.

---

## Skipping Elements

### Definitions

**Core Definition:** Skipping elements in array destructuring uses elisions (consecutive commas) to ignore values at specific positions.

**Technical Definition:** Per ECMA-262, an `Elision` in an `ArrayAssignmentPattern` is an empty element. During evaluation, the iterator is advanced but no binding is created for that position.

**Beginner-Friendly Explanation:** Skipping lets you ignore positions you don't care about. Just use a comma without a variable name.

### Purposes

- To extract only specific positions.
- To avoid naming unused values.
- To reach later elements without intermediate variables.

### Syntax Rules and Structure

```js
const [, b] = array;        // Skip first element
const [, , c] = array;      // Skip first two
const [first, , third] = arr; // Skip second
```

**Syntax Rules:**
- Each comma represents a position.
- An empty position is an elision (skip).
- Trailing commas are ignored (do not create additional positions).
- Can be combined with defaults, rest, and nesting.

**Constraints and Limitations:**
- Each skip advances the iterator, which has side effects for generators.
- Cannot skip with a name unless you use an unused variable.

### Annotated Code Examples

```js
const arr = [1, 2, 3, 4, 5];

// Step 1: Skip the first element.
const [, second] = arr;
console.log(second); // 2

// Step 2: Skip multiple.
const [, , third] = arr;
console.log(third); // 3

// Step 3: Extract first and last.
const [first, , , , fifth] = arr;
console.log(first, fifth); // 1 5

// Step 4: Combined with rest.
const [, second2, ...rest] = arr;
console.log(second2); // 2
console.log(rest);    // [3, 4, 5]
```

**Expected Output:**
```
2
3
1 5
2
[ 3, 4, 5 ]
```

**Why This Output:** Each comma skips a position; the rest element collects the remaining values.

### Real-World Cases

- **API responses:** `const [, data] = response;` (skip status).
- **Coordinate pairs:** `const [, y] = point;`.
- **Event handlers:** `const [, event] = args;`.

---

## Rest Elements

### Definitions

**Core Definition:** The rest element in array destructuring collects all remaining elements into a new array.

**Technical Definition:** Per ES2015, the `...rest` syntax in an array pattern creates a new array containing all elements from the current position to the end of the source. The rest target must be a valid binding identifier and must be the last element in the pattern.

**Beginner-Friendly Explanation:** Rest lets you say "give me these first few, and put everything else in a new array."

### Purposes

- To extract the first elements while keeping the rest together.
- To implement head/tail decomposition.
- To collect variable-length arguments.
- To split arrays into parts.

### Syntax Rules and Structure

```js
const [first, ...rest] = array;
const [a, b, ...others] = array;
```

**Component Breakdown:**
- `first`, `a`, `b`: Named elements.
- `...rest`: Collects the remaining elements.
- Must be the last element.

**Syntax Rules:**
- Must be last in the pattern.
- Creates a new array.
- Can be empty if no elements remain.
- Cannot have a default value.
- Works with any iterable.

**Constraints and Limitations:**
- Must be last.
- Cannot be combined with a default.
- Creates a new array each time.

### Annotated Code Examples

```js
const nums = [1, 2, 3, 4, 5];

// Step 1: Head/tail.
const [head, ...tail] = nums;
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// Step 2: First two, rest.
const [a, b, ...rest] = nums;
console.log(a, b);  // 1 2
console.log(rest);  // [3, 4, 5]

// Step 3: Empty rest.
const [x, ...empty] = [1];
console.log(empty); // []

// Step 4: Rest in function parameters (rest parameters).
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
console.log(sum(1, 2, 3, 4)); // 10
```

**Expected Output:**
```
1
[ 2, 3, 4, 5 ]
1 2
[ 3, 4, 5 ]
[]
10
```

**Why This Output:** Rest collects the remaining elements into a new array; if none remain, it's empty.

### Real-World Cases

- **Head/tail processing:** Recursive algorithms.
- **Skipping the first element:** `const [, ...data] = response;`.
- **Rest parameters:** `function f(first, ...rest) {}`.
- **React:** `const [first, ...others] = items;`.

---

## Nested Destructuring

### Definitions

**Core Definition:** Nested destructuring extracts values from arrays inside arrays using a pattern that mirrors the nested structure.

**Technical Definition:** Per ECMA-262, an `AssignmentElement` can contain a nested `ArrayAssignmentPattern` or `ObjectAssignmentPattern`. During evaluation, the outer value is obtained, then the inner pattern is applied to it.

**Beginner-Friendly Explanation:** Nested destructuring lets you pull values out of arrays inside arrays — like unpacking a suitcase with smaller bags inside.

### Purposes

- To extract deeply nested values in one expression.
- To avoid intermediate variables.
- To destructure complex data structures.
- To handle matrix and coordinate data.

### Syntax Rules and Structure

```js
const [[a, b], [c, d]] = nestedArray;
const [x, [y, z]] = array;
```

**Component Breakdown:**
- Outer `[ ]`: Matches the outer array.
- Inner `[ ]`: Matches nested arrays.
- Values are extracted recursively.

**Syntax Rules:**
- Can mix array and object patterns.
- Can combine with defaults, rest, and skipping.
- Missing intermediates cause `TypeError` unless guarded.
- Each level uses the iterator protocol.

**Constraints and Limitations:**
- Accessing a nested property of `undefined` throws.
- Deeply nested patterns reduce readability.
- Guard with defaults at each level.

### Annotated Code Examples

#### Example 1: Basic Nested Destructuring

```js
const matrix = [[1, 2], [3, 4]];

// Step 1: Destructure nested arrays.
const [[a, b], [c, d]] = matrix;

console.log(a, b); // 1 2
console.log(c, d); // 3 4
```

**Expected Output:**
```
1 2
3 4
```

**Why This Output:** The outer pattern matches the two inner arrays; the inner patterns extract values from each.

#### Example 2: Mixed with Skipping and Defaults

```js
const data = [[1, 2, 3], [4, 5]];

const [[first, , third], [fourth = 0, fifth = 0, sixth = 99]] = data;
console.log(first, third);   // 1 3
console.log(fourth, fifth);  // 4 5
console.log(sixth);          // 99 (default)
```

**Expected Output:**
```
1 3
4 5
99
```

**Why This Output:** Nested patterns can use all destructuring features.

#### Example 3: Mixed Array and Object Destructuring

```js
const response = [
  { id: 1, user: { name: "Alice" } },
  { id: 2, user: { name: "Bob" } },
];

const [{ id: id1, user: { name: name1 } }, { id: id2, user: { name: name2 } }] = response;
console.log(id1, name1); // 1 "Alice"
console.log(id2, name2); // 2 "Bob"
```

**Expected Output:**
```
1 Alice
2 Bob
```

**Why This Output:** Array and object patterns can be nested freely.

### Real-World Cases

- **Matrix operations:** Extracting rows and columns.
- **API responses:** Nested data structures.
- **React props:** Destructuring nested props.
- **Coordinate systems:** `[x, [y, z]]`.

---

## Swapping Variables

### Definitions

**Core Definition:** Array destructuring enables swapping the values of two or more variables without a temporary variable.

**Technical Definition:** Per ECMA-262, the assignment `[a, b] = [b, a]` evaluates the right-hand side first (creating a new array `[b, a]`), then destructures it into `a` and `b`. This works because the right side is fully evaluated before assignment.

**Beginner-Friendly Explanation:** Destructuring lets you swap two variables in one line, without needing a temporary variable.

### Purposes

- To swap variable values concisely.
- To avoid temporary variables.
- To reorder array elements.
- To implement sorting swaps.

### Syntax Rules and Structure

```js
[a, b] = [b, a];
[a, b, c] = [c, a, b];
```

**Syntax Rules:**
- Right-hand side is evaluated first.
- Works with `let` variables (not `const`).
- Can swap multiple variables at once.
- Works with array elements: `[arr[i], arr[j]] = [arr[j], arr[i]]`.

**Constraints and Limitations:**
- Cannot swap `const` bindings.
- Creates a temporary array (minor overhead).

### Annotated Code Examples

#### Example 1: Basic Swap

```js
let a = 1;
let b = 2;

// Step 1: Swap using destructuring.
[a, b] = [b, a];

console.log(a); // 2
console.log(b); // 1
```

**Expected Output:**
```
2
1
```

**Why This Output:** `[b, a]` is evaluated to `[2, 1]`, then destructured into `[a, b]`, swapping the values.

#### Example 2: Swapping Array Elements

```js
const arr = [1, 2, 3, 4, 5];

// Step 1: Swap first and last.
[arr[0], arr[4]] = [arr[4], arr[0]];
console.log(arr); // [5, 2, 3, 4, 1]
```

**Expected Output:**
```
[ 5, 2, 3, 4, 1 ]
```

**Why This Output:** The elements at indices 0 and 4 are swapped in place.

#### Example 3: Rotating Three Variables

```js
let a = 1, b = 2, c = 3;

// Rotate: a <- c, b <- a, c <- b
[a, b, c] = [c, a, b];
console.log(a, b, c); // 3 1 2
```

**Expected Output:**
```
3 1 2
```

**Why This Output:** The right side evaluates to `[3, 1, 2]`, then assigns to `[a, b, c]`.

### Real-World Cases

- **Sorting algorithms:** Swap elements in bubble sort.
- **Game state:** Swap player positions.
- **UI:** Swap item order.
- **Algorithm implementation:** Partitioning, shuffling.

---

## Destructuring in Parameters

### Definitions

**Core Definition:** Destructuring in parameters allows a function to receive an array and immediately unpack its elements into named parameters.

**Technical Definition:** Per ECMA-262, a `BindingElement` in a function's `FormalParameters` can be an `ArrayBindingPattern`. When the function is called, the argument is destructured according to the pattern, and the extracted values are bound to local parameters.

**Beginner-Friendly Explanation:** Instead of accepting an array and then unpacking it inside the function, you can unpack it right in the parameter list.

### Purposes

- To make function signatures self-documenting.
- To avoid repetitive index access.
- To provide default values for parameters.
- To enable tuple-like parameter handling.
- To destructure return values from functions.

### Syntax Rules and Structure

```js
function fn([a, b]) { ... }
function fn([a, b] = []) { ... }
const fn = ([a, b]) => { ... };
```

**Component Breakdown:**
- `[a, b]`: Destructuring pattern as parameter.
- `= []`: Default for the entire parameter.
- Inner defaults: `a = defaultA`.

**Syntax Rules:**
- The parameter must be an iterable.
- Calling without arguments when no `= []` default throws.
- Can combine with renaming (for objects), nesting, and rest.
- Arrow functions support parameter destructuring.

**Constraints and Limitations:**
- Without a `= []` default, calling with no argument throws.
- Deeply nested patterns reduce readability.
- Destructuring in parameters creates new bindings.

### Annotated Code Examples

#### Example 1: Basic Parameter Destructuring

```js
// Step 1: Define a function with destructured parameters.
function distance([x1, y1], [x2, y2]) {
  const dx = x2 - x1;
  const dy = y2 - y1;
  return Math.sqrt(dx * dx + dy * dy);
}

console.log(distance([0, 0], [3, 4])); // 5
```

**Expected Output:**
```
5
```

**Why This Output:** Each array argument is destructured into `x` and `y` components.

#### Example 2: Default Parameter Array

```js
function createPoint([x = 0, y = 0] = []) {
  return { x, y };
}

console.log(createPoint());         // { x: 0, y: 0 }
console.log(createPoint([5]));      // { x: 5, y: 0 }
console.log(createPoint([5, 10]));  // { x: 5, y: 10 }
```

**Expected Output:**
```
{ x: 0, y: 0 }
{ x: 5, y: 0 }
{ x: 5, y: 10 }
```

**Why This Output:** The `= []` default allows calling without arguments; inner defaults fill missing elements.

#### Example 3: Arrow Function Destructuring

```js
const format = ([first, last]) => `${last}, ${first}`;
console.log(format(["Alice", "Smith"])); // "Smith, Alice"
```

**Expected Output:**
```
Smith, Alice
```

**Why This Output:** Arrow functions support destructured parameters directly.

#### Example 4: Rest Parameters and Destructuring

```js
function head([first, ...rest]) {
  return { first, rest };
}

console.log(head([1, 2, 3, 4])); // { first: 1, rest: [2, 3, 4] }
```

**Expected Output:**
```
{ first: 1, rest: [ 2, 3, 4 ] }
```

**Why This Output:** Rest elements work in destructured parameters.

### Real-World Cases

- **React hooks:** `const [value, setValue] = useState();`.
- **Math utilities:** `distance([x1, y1], [x2, y2])`.
- **Tuple-like data:** Coordinates, RGB values.
- **Pipeline functions:** `map(([k, v]) => ...)`.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Destructuring Assignment — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-destructuring-assignment
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array Destructuring — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-runtime-semantics-destructuringassignmentevaluation
- MDN Web Docs – Destructuring assignment — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
- MDN Web Docs – Array destructuring — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#array_destructuring
- MDN Web Docs – Default values — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#default_value
- MDN Web Docs – Swapping variables — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#swapping_variables
- MDN Web Docs – Parsing an array returned from a function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#parsing_an_array_returned_from_a_function
- MDN Web Docs – Ignoring some returned values — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#ignoring_some_returned_values
- MDN Web Docs – Rest parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters
- MDN Web Docs – Array.prototype.entries() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/entries
- React Docs – useState — https://react.dev/reference/react/useState
- Exploring JS – Destructuring — https://exploringjs.com/impatient-js/ch_destructuring.html
- TC39 Proposal – Destructuring Binding — https://github.com/tc39/proposal-destructuring-binding
- You Don't Know JS – Destructuring — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/es-next&beyond/ch3.md
- JavaScript.info – Destructuring assignment — https://javascript.info/destructuring-assignment