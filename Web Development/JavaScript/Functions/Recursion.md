# Recursion — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Recursion is a programming technique in which a function calls itself, either directly or indirectly, to solve a problem by breaking it down into smaller instances of the same problem.

**Technical Definition:** Per MDN, "Recursion is the act of a function calling itself. A recursive function can receive two inputs: a base case (ends recursion) or a recursive case (resumes recursion)". In ECMAScript, recursion is not a special syntax but a natural consequence of functions being first-class values that can reference themselves through their binding identifier or the `arguments.callee` property (deprecated in strict mode). Each recursive call creates a new execution context and a new stack frame, consuming call-stack space until the base case is reached.

**Beginner-Friendly Explanation:** Recursion is when a function calls itself to solve a problem. It's like Russian nesting dolls — you open one doll, find a smaller doll inside, open that one, and keep going until you reach the smallest doll (the base case). Each recursive call works on a slightly smaller version of the problem until the problem is so small it can be answered directly.

### Key Characteristics

- **Self-reference:** A recursive function calls itself, directly or through mutual recursion.
- **Base case required:** Every recursive function must have at least one termination condition to prevent infinite recursion.
- **Stack-based execution:** Each recursive call creates a new stack frame; deep recursion can exhaust the call stack.
- **Divide and conquer:** Recursion is naturally suited to problems that can be decomposed into smaller subproblems of the same type.
- **Elegance vs. efficiency:** Recursive solutions are often more concise and readable but may be slower and more memory-intensive than iterative equivalents.
- **Tail call optimisation (TCO):** ES2015 specifies TCO for strict-mode tail calls, but engine support is inconsistent.

### Prerequisites

- Understanding of JavaScript functions (declarations, expressions, arrow functions).
- Familiarity with the call stack and execution contexts.
- Basic knowledge of conditionals and loops.
- Awareness of scope and closures.

### Related Programming Areas

- **Data structures:** Trees, linked lists, graphs, and nested objects.
- **Algorithms:** Divide and conquer, dynamic programming, backtracking, and depth-first search.
- **Functional programming:** Recursion is a fundamental control-flow mechanism in functional languages.
- **Compiler design:** Recursive descent parsers and abstract syntax tree traversal.

### Core Concepts / Features

---

## Recursive Functions

### Definitions

**Core Definition:** A recursive function is a function that calls itself, either directly or indirectly, to solve a problem.

**Technical Definition:** In ECMAScript, a recursive function is any function whose body contains a `CallExpression` that resolves to the function itself (direct recursion) or to another function that eventually calls it back (indirect or mutual recursion). Each invocation creates a new `Execution Context` that is pushed onto the call stack. The function must have at least one base case that returns without making a recursive call; otherwise, infinite recursion occurs, eventually throwing a `RangeError: Maximum call stack size exceeded`.

**Beginner-Friendly Explanation:** A recursive function is a function that solves a problem by calling itself with a simpler version of the problem. For example, to calculate the factorial of 5 (5!), you can say: 5! = 5 × 4!, and 4! = 4 × 3!, and so on, until you reach 1! = 1. Each step makes the problem smaller until you hit the base case.

### Purposes

- To solve problems that can be naturally decomposed into smaller, identical subproblems.
- To traverse recursive data structures such as trees and graphs.
- To implement divide-and-conquer algorithms elegantly.
- To write concise, readable code for mathematically recursive definitions.
- To model processes that are inherently recursive (e.g., parsing nested expressions).

### Syntax Rules and Structure

```js
function recursiveFunction(input) {
  // Base case: terminates recursion
  if (baseCondition(input)) {
    return baseValue;
  }
  // Recursive case: calls itself with a smaller input
  return recursiveFunction(smallerInput(input));
}
```

**Component Breakdown:**
- `baseCondition(input)`: The condition that determines when recursion stops.
- `baseValue`: The value returned when the base case is met.
- `recursiveFunction(smallerInput(input))`: The recursive call with a reduced input.
- Each recursive call must make progress toward the base case (e.g., decrement a counter, traverse a child node).

**Syntax Rules:**
- A recursive function must have at least one base case.
- The recursive call must be reachable and must eventually lead to the base case.
- The function must not be an arrow function if it needs to reference itself by name (arrow functions have no own `name` binding for recursion unless assigned to a variable).
- In strict mode, `arguments.callee` is not available; use a named function expression or a variable reference.

**Constraints and Limitations:**
- Deep recursion can cause a `RangeError: Maximum call stack size exceeded`.
- Recursion is generally slower than iteration due to function-call overhead and stack management.
- Without TCO, each recursive call adds a new stack frame.

### Annotated Code Examples

#### Example 1: Factorial (Direct Recursion)

```js
// Step 1: Define a recursive factorial function.
function factorial(n) {
  // Step 2: Base case — 0! and 1! are both 1.
  if (n <= 1) {
    return 1;
  }
  // Step 3: Recursive case — n! = n × (n-1)!
  return n * factorial(n - 1);
}

// Step 4: Test the function.
console.log(factorial(5));  // 120 (5 × 4 × 3 × 2 × 1)
console.log(factorial(10)); // 3628800
```

**Expected Output:**
```
120
3628800
```

**Why This Output:** `factorial(5)` calls `factorial(4)`, which calls `factorial(3)`, and so on until `factorial(1)` returns `1`. The results are multiplied as the stack unwinds: `1 × 2 × 3 × 4 × 5 = 120`.

#### Example 2: Mutual Recursion

```js
// Step 1: Define two functions that call each other.
function isEven(n) {
  if (n === 0) return true;
  return isOdd(n - 1);
}

function isOdd(n) {
  if (n === 0) return false;
  return isEven(n - 1);
}

// Step 2: Test mutual recursion.
console.log(isEven(10)); // true
console.log(isOdd(7));   // true
console.log(isEven(3));  // false
```

**Expected Output:**
```
true
true
false
```

**Why This Output:** `isEven(10)` calls `isOdd(9)`, which calls `isEven(8)`, and so on. The functions alternate until `n` reaches `0`. Mutual recursion works because both functions are hoisted (as function declarations) and can reference each other.

### Real-World Cases

- **Mathematical computations:** Factorials, Fibonacci sequences, and combinatorics.
- **Tree traversal:** Recursively visiting nodes in a DOM tree, file system, or organisation chart.
- **Parsing:** Recursive descent parsers for programming languages and expression evaluators.
- **Backtracking algorithms:** Solving mazes, the N-Queens problem, and Sudoku.

---

## Base Case

### Definitions

**Core Definition:** A base case is the condition in a recursive function that terminates the recursion and returns a value without making another recursive call.

**Technical Definition:** The base case is a `return` statement guarded by a conditional expression that evaluates to `true` for the smallest instance of the problem. Per MDN, "A recursive function can receive two inputs: a base case (ends recursion) or a recursive case (resumes recursion)". Without a base case, the function recurses indefinitely, leading to a stack overflow. The base case is also called the "termination condition" or "stopping condition."

**Beginner-Friendly Explanation:** The base case is the "smallest version" of the problem that you can answer directly without further recursion. It's the signal to stop. For example, in a factorial function, the base case is `n <= 1` — you don't need to recurse any further because 1! is just 1.

### Purposes

- To terminate recursion and prevent infinite loops.
- To provide a directly computable answer for the simplest instance of the problem.
- To serve as the foundation upon which all larger solutions are built.
- To define the boundary condition for recursive data structure traversal.

### Syntax Rules and Structure

```js
function recursiveFunction(input) {
  // Base case (must come first or be reachable):
  if (baseCondition(input)) {
    return baseValue;
  }
  // Recursive case:
  return recursiveFunction(smallerInput(input));
}
```

**Component Breakdown:**
- `baseCondition(input)`: A boolean expression that tests whether the problem is small enough to solve directly.
- `baseValue`: The value returned for the simplest case.

**Syntax Rules:**
- The base case must be checked before the recursive call.
- There may be multiple base cases (e.g., `n === 0` and `n === 1`).
- The base case must be reachable from every recursive path.

**Constraints and Limitations:**
- Missing or unreachable base cases cause infinite recursion and stack overflow.
- Base cases that are too broad may skip necessary computation.
- Base cases that are too narrow may require more recursive calls than necessary.

### Annotated Code Examples

#### Example 1: Base Case in Factorial

```js
function factorial(n) {
  // Base case: 0! and 1! are defined as 1.
  if (n <= 1) {
    return 1;
  }
  // Recursive case:
  return n * factorial(n - 1);
}

console.log(factorial(0)); // 1 (base case)
console.log(factorial(1)); // 1 (base case)
console.log(factorial(5)); // 120
```

**Expected Output:**
```
1
1
120
```

**Why This Output:** When `n` is `0` or `1`, the base case returns `1` immediately, preventing further recursion. For `n = 5`, the recursion unwinds until the base case is reached.

#### Example 2: Multiple Base Cases

```js
function fibonacci(n) {
  // Base cases: fib(0) = 0, fib(1) = 1
  if (n === 0) return 0;
  if (n === 1) return 1;
  // Recursive case:
  return fibonacci(n - 1) + fibonacci(n - 2);
}

console.log(fibonacci(0));  // 0
console.log(fibonacci(1));  // 1
console.log(fibonacci(10)); // 55
```

**Expected Output:**
```
0
1
55
```

**Why This Output:** The Fibonacci sequence has two base cases: `fib(0) = 0` and `fib(1) = 1`. These provide the starting values for the recursive computation.

### Real-World Cases

- **Mathematical sequences:** Factorials, Fibonacci numbers, and Pascal's triangle.
- **Tree traversal:** The base case is a leaf node (a node with no children).
- **Graph traversal:** The base case is a visited node or a node with no unvisited neighbours.
- **String processing:** The base case is an empty string or a single character.

---

## Recursive Case

### Definitions

**Core Definition:** The recursive case is the part of a recursive function that calls the function again with a smaller or simpler input, moving toward the base case.

**Technical Definition:** The recursive case is the code path that is executed when the base case condition is not met. It typically involves calling the function with modified arguments that represent a smaller subproblem. Per MDN, "a recursive case (resumes recursion)". Each recursive call must make progress toward the base case; otherwise, infinite recursion occurs.

**Beginner-Friendly Explanation:** The recursive case is where the function says, "I can't solve this directly, but I can solve a smaller version of it and use that to build the answer." For example, in factorial, the recursive case is `n * factorial(n - 1)` — it solves the smaller problem `(n-1)!` and multiplies by `n`.

### Purposes

- To decompose a problem into smaller subproblems of the same type.
- To leverage the solution of a smaller instance to solve a larger instance.
- To implement divide-and-conquer and backtracking algorithms.
- To traverse recursive data structures by visiting children.

### Syntax Rules and Structure

```js
function recursiveFunction(input) {
  if (baseCondition(input)) {
    return baseValue;
  }
  // Recursive case:
  return combine(input, recursiveFunction(smallerInput(input)));
}
```

**Component Breakdown:**
- `smallerInput(input)`: A transformation that reduces the problem size.
- `combine(input, result)`: An operation that combines the current input with the result of the recursive call.
- The recursive call must be in a position where its result is used to compute the final result.

**Syntax Rules:**
- The recursive case must be reachable when the base case is not met.
- The recursive call must use arguments that make progress toward the base case.
- The recursive case may involve multiple recursive calls (e.g., tree traversal visits both left and right children).

**Constraints and Limitations:**
- The recursive case must not create an infinite loop by calling with the same arguments.
- Multiple recursive calls can lead to exponential time complexity if not optimised (e.g., naive Fibonacci).

### Annotated Code Examples

#### Example 1: Recursive Case in Factorial

```js
function factorial(n) {
  if (n <= 1) return 1;         // Base case
  return n * factorial(n - 1);  // Recursive case
}

console.log(factorial(4)); // 24
```

**Expected Output:**
```
24
```

**Why This Output:** The recursive case multiplies `n` by the result of `factorial(n - 1)`. For `n = 4`, this is `4 × factorial(3)`, which is `4 × 3 × 2 × 1 = 24`.

#### Example 2: Recursive Case in Tree Traversal

```js
// Step 1: Define a tree node structure.
const tree = {
  value: 1,
  children: [
    { value: 2, children: [] },
    { value: 3, children: [{ value: 4, children: [] }] },
  ],
};

// Step 2: Recursive traversal function.
function sumTree(node) {
  // Step 3: Base case — leaf node with no children.
  if (node.children.length === 0) {
    return node.value;
  }
  // Step 4: Recursive case — sum the current node and all children.
  let total = node.value;
  for (const child of node.children) {
    total += sumTree(child); // Recursive call for each child
  }
  return total;
}

console.log(sumTree(tree)); // 10 (1 + 2 + 3 + 4)
```

**Expected Output:**
```
10
```

**Why This Output:** The recursive case iterates over each child and recursively sums their subtrees. The base case returns the leaf node's value. The total is the sum of all nodes.

### Real-World Cases

- **Divide and conquer:** Merge sort, quicksort, and binary search.
- **Tree traversal:** Pre-order, in-order, and post-order traversal of binary trees.
- **Graph traversal:** Depth-first search (DFS) of graphs.
- **Backtracking:** Solving puzzles like Sudoku, N-Queens, and mazes.

---

## Stack Overflow

### Definitions

**Core Definition:** A stack overflow is an error that occurs when the call stack exceeds its maximum allowed size, typically caused by infinite or excessively deep recursion.

**Technical Definition:** Per MDN, the error is a `RangeError: Maximum call stack size exceeded` (in Chrome/V8) or `InternalError: too much recursion` (in Firefox/SpiderMonkey). Each function call pushes a new stack frame onto the call stack. When the stack's memory limit is reached, the engine throws an error and halts execution. Browser-specific stack limits are approximately: Chrome/Chromium ~11,000 calls, Firefox ~26,000 calls, Safari ~45,000 calls, and Internet Explorer ~3,000 calls. These limits vary based on available memory and system resources.

**Beginner-Friendly Explanation:** A stack overflow happens when your recursive function keeps calling itself without stopping, and the browser runs out of "stack space" to keep track of all the calls. It's like trying to stack too many books — eventually, the stack falls over. To fix it, make sure your function has a base case that stops the recursion.

### Purposes

- To signal that recursion has exceeded the engine's call stack limit.
- To prevent the browser from freezing or crashing due to runaway recursion.
- To alert the developer to a missing or unreachable base case.
- To enforce a safety boundary on recursive depth.

### Syntax Rules and Structure

Stack overflow is a runtime error, not a syntax construct. The relevant pattern is:

```js
function infiniteRecursion() {
  // No base case — infinite recursion
  infiniteRecursion();
}

infiniteRecursion(); // RangeError: Maximum call stack size exceeded
```

**Component Breakdown:**
- Each call to `infiniteRecursion()` pushes a new frame.
- When the stack limit is reached, the engine throws a `RangeError`.
- The error message varies by engine.

**Syntax Rules:**
- Stack overflow is thrown as a `RangeError` (Chrome) or `InternalError` (Firefox).
- The error can be caught with `try...catch`, but catching may be unreliable at stack-exhaustion depths.
- The maximum call stack size can be measured with a recursive function that catches the error.

**Constraints and Limitations:**
- Stack limits are implementation-dependent and vary across browsers and environments.
- Deep but finite recursion can still overflow if the input is large enough.
- Asynchronous recursion does not overflow the call stack in the same way (each async operation creates a new microtask).

### Annotated Code Examples

#### Example 1: Measuring the Call Stack Size

```js
// Step 1: Define a function that measures the maximum call stack size.
function getMaxCallStackSize(i) {
  try {
    return getMaxCallStackSize(++i);
  } catch {
    return i;
  }
}

// Step 2: Log the result.
console.log(getMaxCallStackSize(0)); // e.g., 10987 (varies by engine)
```

**Expected Output (approximate):**
```
10987
```

**Why This Output:** The function recurses until the stack overflows. The `catch` block captures the `RangeError` and returns the current depth, which is the maximum call stack size for that environment.

#### Example 2: Stack Overflow from Missing Base Case

```js
function countdown(n) {
  console.log(n);
  countdown(n - 1); // No base case — will recurse forever
}

try {
  countdown(5);
} catch (err) {
  console.log(err.name);    // "RangeError"
  console.log(err.message); // "Maximum call stack size exceeded"
}
```

**Expected Output:**
```
5
4
3
2
1
0
-1
... (continues until stack overflow)
RangeError
Maximum call stack size exceeded
```

**Why This Output:** The function lacks a base case. It recurses infinitely, printing decreasing numbers until the call stack is exhausted. The `catch` block captures the `RangeError`.

### Real-World Cases

- **Deep DOM trees:** Recursive traversal of deeply nested DOM nodes can overflow.
- **Large data sets:** Recursive processing of arrays or objects with thousands of elements.
- **Infinite loops in frameworks:** React `useEffect` loops that update state without proper dependencies.
- **Mutual recursion bugs:** Two functions that call each other without a base case.

---

## Tail Recursion

### Definitions

**Core Definition:** Tail recursion is a special case of recursion where the recursive call is the last operation performed by the function, with no additional computation after the call returns.

**Technical Definition:** Per Exploring JS, "Whenever the last thing a function does is to call another function then the latter does not need to return to its caller. As a consequence, no information needs to be stored on the call stack and the function call is more of a goto". A tail call is a function call in tail position — the final action of a function. In a tail-recursive function, the recursive call is in tail position, meaning its result is returned directly without further processing.

**Beginner-Friendly Explanation:** Tail recursion is when the recursive call is the very last thing the function does. Nothing happens after the recursive call returns — its result is immediately returned. This is important because, in languages that support tail call optimisation, the engine can reuse the same stack frame instead of creating a new one, preventing stack overflow.

### Purposes

- To enable tail call optimisation, which eliminates stack growth.
- To write recursive functions that can handle large inputs without stack overflow.
- To transform non-tail-recursive functions into tail-recursive form using accumulators.
- To improve performance by reducing stack frame allocation.

### Syntax Rules and Structure

```js
// Non-tail-recursive factorial:
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // Multiplication after the call — NOT tail
}

// Tail-recursive factorial with accumulator:
function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, n * acc); // Recursive call is the last operation
}
```

**Component Breakdown:**
- `acc`: The accumulator that carries the intermediate result.
- In the tail-recursive version, the recursive call is the last operation; its result is returned directly.
- In the non-tail version, `n *` is applied after the recursive call returns.

**Syntax Rules:**
- A call is in tail position if it is the last operation in the function body (directly returned).
- The recursive call must be returned directly: `return f(...)`.
- Using an accumulator is a common technique to convert non-tail recursion to tail recursion.

**Constraints and Limitations:**
- JavaScript engines (V8, SpiderMonkey, JavaScriptCore) do not automatically optimise tail recursion, so tail recursion does not prevent stack overflow in practice.
- Tail call optimisation (TCO) is specified in ES2015 for strict mode but is not widely implemented.
- Tail-recursive code is often less readable than simple loops.

### Annotated Code Examples

#### Example 1: Non-Tail vs Tail-Recursive Factorial

```js
// Non-tail-recursive:
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // Not tail: multiplication after call
}

// Tail-recursive:
function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, n * acc); // Tail: call is the last operation
}

console.log(factorial(5));      // 120
console.log(factorialTail(5));  // 120
```

**Expected Output:**
```
120
120
```

**Why This Output:** Both functions compute the same result. The tail-recursive version passes the intermediate product in the `acc` parameter, so no multiplication is needed after the recursive call returns. This is the tail-recursive form.

#### Example 2: Converting Sum to Tail Recursion

```js
// Non-tail-recursive sum:
function sum(n) {
  if (n === 1) return 1;
  return n + sum(n - 1); // Addition after the call
}

// Tail-recursive sum with accumulator:
function sumTail(n, acc = 0) {
  if (n === 0) return acc;
  return sumTail(n - 1, acc + n); // Call is the last operation
}

console.log(sum(100));     // 5050
console.log(sumTail(100)); // 5050
```

**Expected Output:**
```
5050
5050
```

**Why This Output:** The tail-recursive version accumulates the sum in the `acc` parameter. The recursive call is the final operation, making it a tail call.

### Real-World Cases

- **Functional programming:** Tail recursion is a common idiom in Scheme, Haskell, and other functional languages.
- **Large data processing:** Tail-recursive functions can handle large inputs when TCO is available.
- **State machines:** Tail-recursive implementations of state transitions.
- **Compiler optimisation:** Tail calls are optimised in some JavaScript engines for specific patterns.

---

## Tail Call Optimization

### Definitions

**Core Definition:** Tail call optimisation (TCO) is a language feature that allows a tail call to be executed without growing the call stack, effectively turning recursion into iteration.

**Technical Definition:** Per Exploring JS, "ECMAScript 6 offers tail call optimization, where you can make some function calls without growing the call stack". In ES2015, TCO is specified for calls in tail position, but only in strict mode. When TCO is applied, the current stack frame is replaced by the frame of the called function, rather than adding a new frame. However, most JavaScript engines (including V8, which powers Chrome and Node.js) do not implement TCO, so tail-recursive functions still overflow the stack for large inputs.

**Beginner-Friendly Explanation:** Tail call optimisation is a way for JavaScript engines to make tail-recursive functions use less memory. Normally, each function call adds a new "layer" to the call stack. With TCO, the engine can reuse the same layer, so the stack doesn't grow. Unfortunately, most JavaScript engines don't actually do this, so tail recursion doesn't prevent stack overflow in practice.

### Purposes

- To eliminate stack growth for tail-recursive functions.
- To allow recursion to handle arbitrarily large inputs without stack overflow.
- To improve performance by reducing stack frame allocation.
- To enable functional programming idioms in JavaScript.

### Syntax Rules and Structure

TCO is not a syntax feature; it is a runtime optimisation. The relevant requirement is that the call must be in tail position:

```js
"use strict";

function tailRecursive(n, acc = 0) {
  if (n === 0) return acc;
  return tailRecursive(n - 1, acc + n); // Tail call
}
```

**Component Breakdown:**
- `"use strict"`: TCO is specified for strict mode only.
- `return tailRecursive(...)`: The call must be directly returned.
- Any operation after the call (e.g., `n + ...`) disqualifies TCO.

**Syntax Rules:**
- TCO is only specified for strict mode.
- The call must be in tail position: `return f(...)`.
- TCO is not applied to calls in conditional expressions, comma expressions, or other non-tail positions.

**Constraints and Limitations:**
- Most JavaScript engines do not implement TCO.
- V8 (Chrome, Node.js) does not implement TCO and has no plans to do so.
- TCO can make debugging harder because stack frames are missing.
- The TC39 committee has discussed replacing TCO with a syntactic opt-in (Syntactic Tail Calls).

### Annotated Code Examples

#### Example 1: Tail Call in Strict Mode

```js
"use strict";

// Step 1: Define a tail-recursive function.
function sumTail(n, acc = 0) {
  if (n === 0) return acc;
  return sumTail(n - 1, acc + n); // Tail call
}

// Step 2: Call with a large number.
console.log(sumTail(10000)); // 50005000 (may overflow in non-TCO engines)
```

**Expected Output (in engines without TCO):**
```
RangeError: Maximum call stack size exceeded
```

**Expected Output (in engines with TCO):**
```
50005000
```

**Why This Output:** In engines without TCO (most browsers), the tail-recursive function still creates a new stack frame for each call, eventually overflowing. In engines with TCO (rare), the stack frame is reused, allowing the function to complete.

#### Example 2: Non-Tail Call (TCO Not Applied)

```js
"use strict";

function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // Not a tail call — multiplication after
}
```

**Why TCO Does Not Apply:** The recursive call is not in tail position because the multiplication `n *` occurs after the call returns. TCO cannot be applied.

### Real-World Cases

- **Functional languages:** TCO is essential in Scheme, Haskell, and Erlang.
- **JavaScript engines:** TCO is specified but not widely implemented; developers use trampolines or loops instead.
- **Syntactic Tail Calls:** A TC39 proposal to add an explicit `continue` keyword for tail calls.

---

## Recursion vs Iteration

### Definitions

**Core Definition:** Recursion and iteration are two fundamental approaches to repetition. Recursion solves a problem by having a function call itself; iteration repeats a block of code using loops.

**Technical Definition:** Per the javascript.info tutorial, "The loop variant is the second in terms of speed. In both the recursive and the loop variant we sum the same numbers. But the recursion involves nested calls and execution stack management. That also takes resources, so it's slower". Recursion uses the call stack for repetition; iteration uses loop constructs (`for`, `while`, `do...while`). Recursion can express solutions more elegantly for recursive data structures, while iteration is generally faster and more memory-efficient.

**Beginner-Friendly Explanation:** Recursion and iteration both let you repeat things, but in different ways. Recursion is like a function calling itself over and over. Iteration is like a loop that repeats a block of code. Recursion is often easier to read for problems like tree traversal, while loops are faster and use less memory for simple repetitions.

### Purposes

- To choose the most appropriate control-flow mechanism for a given problem.
- To optimise performance by selecting iteration for simple repetitions.
- To improve readability by selecting recursion for recursive data structures.
- To avoid stack overflow by converting deep recursion to iteration.

### Syntax Rules and Structure

```js
// Iteration:
function sumIterative(n) {
  let sum = 0;
  for (let i = 1; i <= n; i++) {
    sum += i;
  }
  return sum;
}

// Recursion:
function sumRecursive(n) {
  if (n === 1) return 1;
  return n + sumRecursive(n - 1);
}
```

**Component Breakdown:**
- **Iteration:** Uses a loop variable (`i`), a condition (`i <= n`), and an accumulator (`sum`).
- **Recursion:** Uses a base case (`n === 1`) and a recursive call (`sumRecursive(n - 1)`).

**Syntax Rules:**
- Iteration uses `for`, `while`, or `do...while` loops.
- Recursion uses function calls.
- Both can solve the same problems, but with different performance and readability trade-offs.

**Constraints and Limitations:**
- Recursion is limited by the call stack size; iteration is limited by memory for loop variables (typically not a concern).
- Recursion is generally slower due to function-call overhead.
- Iteration can be more verbose for recursive data structures.

### Annotated Code Examples

#### Example 1: Sum Comparison

```js
// Iterative sum:
function sumIterative(n) {
  let sum = 0;
  for (let i = 1; i <= n; i++) {
    sum += i;
  }
  return sum;
}

// Recursive sum:
function sumRecursive(n) {
  if (n === 1) return 1;
  return n + sumRecursive(n - 1);
}

console.log(sumIterative(100)); // 5050
console.log(sumRecursive(100)); // 5050
```

**Expected Output:**
```
5050
5050
```

**Why This Output:** Both functions compute the same result. The iterative version uses a loop, while the recursive version uses function calls. For `n = 100`, both are fine, but for very large `n`, the recursive version may overflow.

#### Example 2: Tree Traversal — Recursion Is Natural

```js
// Recursive tree traversal:
function traverseTree(node) {
  console.log(node.value);
  for (const child of node.children) {
    traverseTree(child);
  }
}

// Iterative tree traversal (using a stack):
function traverseTreeIterative(root) {
  const stack = [root];
  while (stack.length > 0) {
    const node = stack.pop();
    console.log(node.value);
    for (let i = node.children.length - 1; i >= 0; i--) {
      stack.push(node.children[i]);
    }
  }
}
```

**Expected Output:** Both produce the same traversal order.

**Why This Output:** Recursion is natural for tree traversal because the structure is recursive. The iterative version requires an explicit stack, which is more verbose but avoids stack overflow.

### Real-World Cases

- **Performance-critical code:** Prefer iteration for simple repetitions.
- **Recursive data structures:** Prefer recursion for trees and graphs.
- **Stack-limited environments:** Use iteration when recursion depth may be large.
- **Code readability:** Use recursion when it expresses the problem more clearly.

---

## Recursive Data Structures

### Definitions

**Core Definition:** A recursive data structure is a data structure that is defined in terms of itself, such as a tree where each node contains references to other nodes of the same type.

**Technical Definition:** Recursive data structures are those in which a node or element contains references to other instances of the same structure. Common examples include linked lists (each node points to the next), trees (each node has child nodes), and graphs (vertices connected by edges). Per the MDN recursion glossary, "Recursion is used to solve problems that contain smaller sub-problems". Recursive data structures naturally lend themselves to recursive algorithms because the structure itself is defined recursively.

**Beginner-Friendly Explanation:** A recursive data structure is a structure that contains smaller versions of itself. A tree is the classic example: a tree is a node with children, and each child is itself a tree. This means you can write a recursive function to process the whole tree by processing each node and then recursively processing its children.

### Purposes

- To model hierarchical relationships naturally (file systems, DOM, organisation charts).
- To enable recursive algorithms that mirror the structure of the data.
- To support efficient insertion, deletion, and traversal operations.
- To represent mathematical structures like expressions and fractals.

### Syntax Rules and Structure

```js
// Tree node structure:
const tree = {
  value: 1,
  children: [
    { value: 2, children: [] },
    { value: 3, children: [{ value: 4, children: [] }] },
  ],
};

// Recursive traversal:
function traverse(node) {
  console.log(node.value);
  for (const child of node.children) {
    traverse(child);
  }
}
```

**Component Breakdown:**
- `node.value`: The data stored in the node.
- `node.children`: An array of child nodes (each a recursive data structure).
- `traverse(node)`: A recursive function that visits the node and all descendants.

**Syntax Rules:**
- Recursive data structures can be built with objects, arrays, or classes.
- The base case is typically a node with no children (a leaf).
- Traversal functions must handle both internal nodes and leaves.

**Constraints and Limitations:**
- Deeply nested structures can cause stack overflow in recursive traversal.
- Cycles (in graphs) require a `visited` set to prevent infinite recursion.
- Memory usage can be high for very large structures.

### Annotated Code Examples

#### Example 1: Binary Tree Traversal

```js
// Step 1: Define a binary tree node.
class TreeNode {
  constructor(value, left = null, right = null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }
}

// Step 2: Build a tree.
const root = new TreeNode(1,
  new TreeNode(2, new TreeNode(4), new TreeNode(5)),
  new TreeNode(3, new TreeNode(6), null)
);

// Step 3: In-order traversal (left, root, right).
function inOrder(node) {
  if (node === null) return; // Base case
  inOrder(node.left);        // Recursive case: left subtree
  console.log(node.value);   // Visit node
  inOrder(node.right);       // Recursive case: right subtree
}

inOrder(root); // 4, 2, 5, 1, 6, 3
```

**Expected Output:**
```
4
2
5
1
6
3
```

**Why This Output:** In-order traversal visits the left subtree, then the node, then the right subtree. The base case (`node === null`) stops recursion at leaf nodes.

#### Example 2: Linked List Traversal

```js
// Step 1: Define a linked list node.
function ListNode(value, next = null) {
  return { value, next };
}

// Step 2: Build a linked list.
const list = ListNode(1, ListNode(2, ListNode(3, null)));

// Step 3: Recursive traversal.
function printList(node) {
  if (node === null) return; // Base case
  console.log(node.value);
  printList(node.next); // Recursive case
}

printList(list); // 1, 2, 3
```

**Expected Output:**
```
1
2
3
```

**Why This Output:** The linked list is a recursive data structure: each node points to the next. The base case is `null` (end of list). The recursive call processes the next node.

### Real-World Cases

- **DOM tree:** HTML elements form a tree that can be traversed recursively.
- **File system:** Directories contain files and subdirectories.
- **JSON objects:** Nested objects and arrays.
- **Abstract syntax trees (ASTs):** Parser output for programming languages.
- **Graphs:** Social networks, road maps, and dependency graphs.

---

## Tree Traversal

### Definitions

**Core Definition:** Tree traversal is the process of visiting every node in a tree data structure exactly once, typically using recursion.

**Technical Definition:** Tree traversal algorithms visit each node in a specific order. The three main depth-first traversals are pre-order (node, left, right), in-order (left, node, right), and post-order (left, right, node). Breadth-first traversal (level-order) visits nodes level by level. Per the data-structure-typed documentation, "Binary Tree DFS | Traverse a binary tree in a depth-first manner, starting from the root node, first visiting the left subtree, and then the right subtree, using recursion". Recursive traversal is natural because a tree is a recursive data structure.

**Beginner-Friendly Explanation:** Tree traversal means visiting every node in a tree. There are different orders: pre-order visits the parent first, in-order visits the left child, then parent, then right child (used for sorted output in binary search trees), and post-order visits children before the parent. Recursion makes traversal easy because each subtree is itself a tree.

### Purposes

- To visit and process every node in a tree exactly once.
- To search for a specific value in a tree.
- To produce a sorted sequence from a binary search tree (in-order traversal).
- To evaluate mathematical expressions (post-order).
- To serialise or copy a tree structure.

### Syntax Rules and Structure

```js
// Pre-order traversal:
function preOrder(node) {
  if (node === null) return;
  console.log(node.value);  // Visit node
  preOrder(node.left);      // Traverse left
  preOrder(node.right);     // Traverse right
}

// In-order traversal:
function inOrder(node) {
  if (node === null) return;
  inOrder(node.left);
  console.log(node.value);
  inOrder(node.right);
}

// Post-order traversal:
function postOrder(node) {
  if (node === null) return;
  postOrder(node.left);
  postOrder(node.right);
  console.log(node.value);
}
```

**Component Breakdown:**
- `node === null`: The base case (empty subtree).
- The order of the recursive calls determines the traversal type.

**Syntax Rules:**
- The base case is typically `node === null` (for binary trees) or `node.children.length === 0` (for n-ary trees).
- The position of the node visit relative to the recursive calls determines the traversal order.

**Constraints and Limitations:**
- Deep trees can cause stack overflow in recursive traversal.
- Recursive traversal is not suitable for trees with cycles (use a `visited` set).

### Annotated Code Examples

#### Example 1: Pre-Order Traversal

```js
class TreeNode {
  constructor(value, left = null, right = null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }
}

const root = new TreeNode(1,
  new TreeNode(2, new TreeNode(4), new TreeNode(5)),
  new TreeNode(3, new TreeNode(6), null)
);

function preOrder(node) {
  if (node === null) return;
  console.log(node.value); // Visit node first
  preOrder(node.left);
  preOrder(node.right);
}

preOrder(root); // 1, 2, 4, 5, 3, 6
```

**Expected Output:**
```
1
2
4
5
3
6
```

**Why This Output:** Pre-order visits the node before its children. The root (1) is printed first, then the left subtree (2, 4, 5), then the right subtree (3, 6).

#### Example 2: In-Order Traversal (Binary Search Tree)

```js
// Binary search tree:
const bst = new TreeNode(5,
  new TreeNode(3, new TreeNode(2), new TreeNode(4)),
  new TreeNode(8, new TreeNode(7), new TreeNode(9))
);

function inOrder(node) {
  if (node === null) return;
  inOrder(node.left);
  console.log(node.value); // Visit node in between
  inOrder(node.right);
}

inOrder(bst); // 2, 3, 4, 5, 7, 8, 9 (sorted order)
```

**Expected Output:**
```
2
3
4
5
7
8
9
```

**Why This Output:** In-order traversal of a binary search tree visits nodes in ascending order. The left subtree (2, 3, 4) is visited first, then the root (5), then the right subtree (7, 8, 9).

### Real-World Cases

- **File system navigation:** Recursively listing directories and files.
- **DOM manipulation:** Traversing HTML elements to find or modify nodes.
- **Expression evaluation:** Post-order traversal evaluates arithmetic expressions.
- **Database indexes:** B-tree traversal for range queries.
- **JSON processing:** Recursively visiting nested objects and arrays.

---

## Graph Traversal

### Definitions

**Core Definition:** Graph traversal is the process of visiting every vertex in a graph, typically using depth-first search (DFS) or breadth-first search (BFS). DFS can be implemented recursively.

**Technical Definition:** Per the data-structure-typed documentation, "Graph DFS | Traverse a graph in a depth-first manner, starting from a given node, exploring along one path as deeply as possible, and backtracking to explore other paths". Recursive DFS uses the call stack to track the path; BFS uses a queue and is inherently iterative. Graphs may contain cycles, so a `visited` set is essential to prevent infinite recursion.

**Beginner-Friendly Explanation:** Graph traversal means visiting all the nodes in a graph. Depth-first search (DFS) goes as deep as possible along one path before backtracking. Recursion makes DFS easy because the call stack naturally handles the backtracking. Breadth-first search (BFS) visits all neighbours first, then their neighbours, and so on — it typically uses a queue, not recursion.

### Purposes

- To visit every vertex in a graph.
- To find a path between two vertices.
- To detect cycles in a graph.
- To find connected components.
- To perform topological sorting.

### Syntax Rules and Structure

```js
// Recursive DFS:
function dfs(graph, node, visited = new Set()) {
  if (visited.has(node)) return; // Base case: already visited
  visited.add(node);
  console.log(node); // Visit node
  for (const neighbor of graph[node]) {
    dfs(graph, neighbor, visited); // Recursive call for each neighbor
  }
}
```

**Component Breakdown:**
- `visited`: A `Set` tracking visited nodes to prevent infinite recursion in cyclic graphs.
- `graph[node]`: The adjacency list for the current node.
- The base case is a node that has already been visited.

**Syntax Rules:**
- A `visited` set is required for graphs with cycles.
- The recursive call is made for each unvisited neighbour.
- DFS can also be implemented iteratively using an explicit stack.

**Constraints and Limitations:**
- Deep graphs can cause stack overflow in recursive DFS.
- BFS is preferred for finding shortest paths in unweighted graphs.
- Recursive DFS is not suitable for very large graphs due to stack limits.

### Annotated Code Examples

#### Example 1: Recursive DFS

```js
// Step 1: Define a graph as an adjacency list.
const graph = {
  A: ["B", "C"],
  B: ["D"],
  C: ["E"],
  D: [],
  E: ["F"],
  F: [],
};

// Step 2: Recursive DFS.
function dfs(node, visited = new Set()) {
  // Step 3: Base case — already visited.
  if (visited.has(node)) return;
  // Step 4: Mark as visited and process.
  visited.add(node);
  console.log(node);
  // Step 5: Recursive case — visit all neighbors.
  for (const neighbor of graph[node]) {
    dfs(neighbor, visited);
  }
}

dfs("A"); // A, B, D, C, E, F
```

**Expected Output:**
```
A
B
D
C
E
F
```

**Why This Output:** DFS starts at A, visits B, then D (B's only neighbour), backtracks to A, visits C, then E, then F. The `visited` set prevents revisiting nodes.

#### Example 2: Iterative BFS (for comparison)

```js
function bfs(graph, start) {
  const visited = new Set();
  const queue = [start];
  visited.add(start);

  while (queue.length > 0) {
    const node = queue.shift();
    console.log(node);
    for (const neighbor of graph[node]) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }
}

bfs(graph, "A"); // A, B, C, D, E, F
```

**Expected Output:**
```
A
B
C
D
E
F
```

**Why This Output:** BFS visits all neighbours of A (B, C) before visiting their neighbours (D, E). It uses a queue instead of recursion.

### Real-World Cases

- **Social networks:** Finding friends of friends, shortest connection paths.
- **Web crawling:** Visiting linked pages.
- **Network routing:** Finding paths between routers.
- **Maze solving:** DFS for path finding, BFS for shortest path.
- **Dependency resolution:** Topological sorting of build dependencies.

---

## References

- MDN Web Docs – Recursion (Glossary) — https://developer.mozilla.org/en-US/docs/Glossary/Recursion
- MDN Web Docs – InternalError: too much recursion — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Too_much_recursion
- Exploring JS – Tail call optimization — https://exploringjs.com/es6/ch_tail-calls.html
- TC39 – Syntactic Tail Calls (STC) Proposal — https://github.com/tc39/proposal-ptc-syntax
- TrackJS – How to fix Maximum call stack size exceeded — https://trackjs.com/javascript-errors/maximum-call-stack-exceeded/
- JavaScript.info – Recursion and stack — https://javascript.info/recursion
- GitHub – JavaScript Recursive Optimization — https://github.com/echo-cool-coding/cool-coding/blob/main/docs/language/javascript/15-javascript-functional-programming/7-javascript-recursive-optimization.mdx
- data-structure-typed – npm Documentation — https://www.npmjs.com/package/data-structure-typed
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- Eloquent JavaScript – Functions (Recursion) — https://eloquentjavascript.net/03_functions.html
- MDN Web Docs – Call stack — https://developer.mozilla.org/en-US/docs/Glossary/Call_stack