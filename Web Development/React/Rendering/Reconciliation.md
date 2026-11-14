# React Reconciliation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Reconciliation is the algorithm React uses to compare a previously rendered element tree with a new one and determine the minimal set of DOM operations needed to make the real DOM match the new tree.

**Technical Definition:** When state or props change, `render()` returns a new tree of React elements. React must figure out how to efficiently update the UI to match the most recent tree. Generic tree-diffing algorithms run in O(n³), making them impractical for UI. React instead implements a heuristic O(n) algorithm based on two assumptions: (1) two elements of different types produce different trees, and (2) the developer can hint at stable child elements with a `key` prop. When root elements have different types, React tears down the old tree and builds a new one from scratch. When DOM elements share the same type, React keeps the underlying DOM node and updates only changed attributes. When component elements share the same type, the instance stays the same, so state is maintained across renders. For children, React iterates over both lists simultaneously; with keys, React uses the key to match children in the original tree with children in the subsequent tree .

**Beginner-Friendly Explanation:** Reconciliation is how React figures out what actually changed on the screen so it can update only the parts that need updating, instead of rebuilding everything from scratch. Imagine you have two versions of a shopping list — today's and yesterday's. Instead of rewriting the whole list, you compare them line by line and only change the lines that are different. React does the same thing with your UI: it compares the old "description" of your interface with the new one, finds the differences, and applies only those changes to the real DOM. The `key` prop is React's way of knowing which list items are the same across renders, so it can move them instead of recreating them.

### Key Characteristics

- **Heuristic O(n):** React's algorithm is linear in the number of elements, made possible by two assumptions about types and keys .
- **Type-driven:** Different root element types cause a full teardown and rebuild; state is destroyed .
- **Same-type preservation:** Same DOM element types keep the DOM node and update only changed attributes .
- **Component identity:** Same component types preserve the component instance and state across renders .
- **Key-first child matching:** With keys, React matches children by key first, position second; without keys, position becomes identity .
- **Tearing down loses state:** Full rebuilds destroy component state and DOM nodes .
- **Keys are scoped and stripped:** Keys are scoped to their sibling set and are consumed by React rather than passed to the component as props .

### Prerequisites

- React function components, state, and props.
- Understanding of JSX and React elements.
- Familiarity with list rendering and the `key` prop.
- Basic knowledge of the Virtual DOM and the render → commit pipeline.

### Related Programming Areas

- **Rendering Performance** — reconciliation determines how much DOM work React performs.
- **Keys and Lists** — the primary developer-facing lever for controlling reconciliation.
- **Fiber Architecture** — the incremental, interruptible engine that executes reconciliation.
- **Component Identity** — how React decides whether to preserve or reset state.

---

## 1. Reconciliation

### Definitions

**Core Definition:** Reconciliation is the process of comparing two element trees and computing the minimal set of changes needed to update the DOM.

**Technical Definition:** Reconciliation is the algorithm React runs when an update occurs. It takes the current element tree and the new element tree produced by rendering, compares them recursively, and produces a list of changes to apply to the DOM. React's implementation is a heuristic O(n) algorithm based on element types and developer-provided keys. The Fiber reconciler, which became the default for React 16 and above, reimplements reconciliation as an incremental, interruptible process .

**Beginner-Friendly Explanation:** Reconciliation is React's comparison step. It looks at what you want the screen to look like now, compares it to what it looked like before, and figures out the smallest number of changes needed.

### Purposes

- To update the DOM efficiently without rebuilding the entire tree.
- To preserve DOM nodes and component state where the identity has not changed.
- To provide a predictable, declarative update model.
- To enable high-performance updates even for large trees.

### Syntax Rules and Structure

**Reconciliation Decision Flow:**

| Situation | Behaviour |
|-----------|-----------|
| Different root element types | Tear down old tree; build new tree from scratch; state is lost |
| Same DOM element type | Keep DOM node; update only changed attributes |
| Same component type | Keep component instance; update props; state is preserved |
| Children with keys | Match by key first, position second |
| Children without keys | Match by position (index) |

**Syntax Rules:**

1. Different types at any level cause a full rebuild of that subtree .
2. Same types preserve the underlying node and component instance .
3. Keys give React a stable identity for children across renders .
4. Reconciliation only compares elements at the same level; it does not move elements across levels .

### Annotated Code Examples

```jsx
// Different root types: full teardown and rebuild
<div>
  <Counter />  {/* State preserved? No — div vs span changes the tree */}
</div>

<span>
  <Counter />  {/* New Counter instance; old state is destroyed */}
</span>
```

**Expected Output:** The second render destroys the old `Counter` and mounts a new one.

**Why this output:** React compares the root elements. `<div>` and `<span>` are different types, so React tears down the old tree and builds a new one from scratch, losing all state .

```jsx
// Same DOM element type: node is kept, attributes are updated
<div className="before" title="stuff" />
<div className="after" title="stuff" />
```

**Expected Output:** React modifies only the `className` on the existing DOM node.

**Why this output:** Both elements are `div` types, so React keeps the DOM node and updates only the changed attribute .

### Real-World Cases

- **Tab switching:** Using the same component type preserves state across tabs; different types reset state.
- **Conditional rendering:** Swapping a `<div>` for a `<span>` resets all child state.
- **List updates:** Reconciliation with stable keys moves DOM nodes instead of recreating them.

### References

- React — Reconciliation — https://ru.react.js.org/docs/reconciliation.html
- React — Reconciliation (Legacy) — https://he.legacy.reactjs.org/docs/reconciliation.html

---

## 2. Diffing Algorithm

### Definitions

**Core Definition:** The diffing algorithm is the specific heuristic React uses to compare two element trees element by element.

**Technical Definition:** The diffing algorithm compares the root elements first. If the types differ, the old tree is torn down and a new tree is built. If the types are the same, React updates attributes and recurses on children. For lists, React iterates over both child arrays simultaneously; with keys, it builds a map of existing children by key and matches new children against it, enabling O(n) insertion, deletion, substitution, and moves. Generic tree-diffing algorithms are O(n³); React's heuristic reduces this to O(n) by making two assumptions about types and keys .

**Beginner-Friendly Explanation:** The diffing algorithm is React's line-by-line comparison. It looks at the top of the tree first. If the top element is a totally different kind of thing, it starts over. If it is the same kind, it checks the details and then moves down to the children.

### Purposes

- To compute the minimal set of DOM operations in linear time.
- To avoid the O(n³) cost of generic tree diffing.
- To provide predictable behaviour based on element types and keys.
- To enable efficient list updates with keyed children.

### Syntax Rules and Structure

**Diffing Decision Table:**

| Comparison | Action |
|------------|--------|
| Different root types | Full teardown and rebuild |
| Same DOM element type | Update changed attributes; recurse on children |
| Same component type | Update props; preserve instance and state; recurse |
| Children with keys | Match by key; perform O(n) insert/delete/move |
| Children without keys | Match by position; index becomes identity |

**Syntax Rules:**

1. React compares elements at the same level only; it does not attempt to move elements across tree levels .
2. Different types at any level trigger a full rebuild of that subtree .
3. Keys enable O(n) list reconciliation via a key map .
4. Without keys, position (index) is the identity .

### Annotated Code Examples

```jsx
// Same component type: instance and state are preserved
<Counter />
<Counter />
```

**Expected Output:** The second render updates props on the existing instance; state is maintained.

**Why this output:** React sees the same component type, so it keeps the instance and updates props rather than remounting .

```jsx
// Keyed children: O(n) matching by key
<ul>
  {items.map(item => <li key={item.id}>{item.text}</li>)}
</ul>
```

**Expected Output:** React matches `li` elements by `item.id`, moving DOM nodes rather than recreating them when the list reorders.

**Why this output:** With keys, React builds a map of existing children by key and matches new children against it, enabling efficient moves .

### Real-World Cases

- **Data tables:** Keyed rows allow sorting and filtering without recreating DOM nodes.
- **Sortable lists:** Keys enable React to move items instead of destroying and recreating them.
- **Conditional UI:** Switching element types resets state, which is sometimes desired and sometimes a bug.

### References

- React — Reconciliation: The Diffing Algorithm — https://ru.react.js.org/docs/reconciliation.html
- React — Reconciliation: Heuristics — https://he.legacy.reactjs.org/docs/reconciliation.html

---

## 3. Key Prop

### Definitions

**Core Definition:** The `key` prop is a special attribute that gives React a stable identity hint for an element within its sibling set.

**Technical Definition:** The `key` prop is consumed by React during reconciliation and stripped from `props` by `createElement`. Inside a set of siblings, React matches new elements to existing fibers by key first, position second. Keys are scoped to the sibling set, not global. A key derived from data identity (e.g., a server ID) lets React move, insert, and delete list items efficiently while preserving component state .

**Beginner-Friendly Explanation:** The `key` prop is a name tag for a list item. React uses the name tag to recognise the same item across renders, so it can move it instead of throwing it away and building a new one.

### Purposes

- To give React a stable identity for list children across renders.
- To enable efficient list reconciliation (O(n) insert/delete/move).
- To preserve component state and DOM state (focus, input values) across reorders.
- To prevent unnecessary re-renders and lost state.

### Syntax Rules and Structure

```jsx
<ul>
  {items.map(item => (
    <li key={item.id}>{item.text}</li>
  ))}
</ul>
```

**Syntax Rules:**

1. Keys must be unique among siblings .
2. Keys must be stable across renders .
3. Keys must be derived from the data's identity (e.g., `item.id`) .
4. Keys are not passed to the component as props; they are stripped from `props` .
5. Keys are scoped to the sibling set, not global .

**Constraints and Limitations:**

- Array indices are not stable identities when the list changes shape .
- Random keys (`Math.random()`) cause full remounts on every render .
- Keys are not a substitute for data identity; they must reflect the actual identity of the item.

### Annotated Code Examples

```jsx
// ✅ Stable, unique key from data
<ul>
  {todos.map(todo => (
    <TodoItem key={todo.id} todo={todo} />
  ))}
</ul>
```

**Expected Output:** React matches each `TodoItem` by `todo.id`, preserving state and minimising DOM operations when the list changes.

**Why this output:** The `key` gives React a stable identity for each item. When the list is reordered or filtered, React moves the existing component instances to their new positions .

```jsx
// ❌ Unstable key: index
<ul>
  {todos.map((todo, index) => (
    <TodoItem key={index} todo={todo} />
  ))}
</ul>
```

**Expected Output:** Inserting an item at the beginning causes React to update every row with the wrong item's props, potentially losing state.

**Why this output:** Index keys renumber on every insertion, deletion, sort, and filter. React matches by position, so item A gets item D's props, item B gets item A's props, and so on .

### Real-World Cases

- **Todo lists:** Stable IDs preserve edit state and input focus across reorders.
- **Chat feeds:** Message IDs preserve scroll position and animation state.
- **Data grids:** Row IDs enable efficient sorting and filtering.

### References

- React — Reconciliation: Keys — https://ru.react.js.org/docs/reconciliation.html
- Keys & List Reconciliation — https://raw.githubusercontent.com/ualiyou/frontend-engineering/b5cc4d0dee1575ccd230354e40462e2654dcfc75/docs/02-rendering-frameworks/react/keys-and-list-reconciliation.md#1
- CoreUI — How to use the key prop in React lists — https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/

---

## 4. Keys and Lists

### Definitions

**Core Definition:** Keys in lists are the mechanism by which React matches list items across renders when the list's shape changes.

**Technical Definition:** When rendering an array of children, React uses the `key` prop to match items between renders. Stable keys let React move, insert, and delete list items efficiently. Without keys, React defaults to matching by position (index), which breaks when the list is reordered, filtered, or items are inserted or deleted .

**Beginner-Friendly Explanation:** When you have a list of things on screen, React needs to know which item is which after the list changes. Keys are how you tell React "this is the same item as before, just in a different position."

### Purposes

- To match list items across renders when the list changes shape.
- To enable efficient moves, insertions, and deletions.
- To preserve item-level state (editing, focus, animation) across reorders.
- To prevent state from being associated with the wrong item.

### Syntax Rules and Structure

```jsx
{items.map(item => (
  <li key={item.id}>{item.name}</li>
))}
```

**Syntax Rules:**

1. Every element in an array must have a `key` .
2. Keys must be unique among siblings .
3. Keys should be stable and derived from data identity .
4. Keys should survive sorting, filtering, and insertion .

### Annotated Code Examples

```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <EditableTodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}
```

**Expected Output:** Editing state follows the correct todo when the list is reordered or items are inserted.

**Why this output:** The stable `todo.id` key tells React which `EditableTodoItem` instance corresponds to which todo, even after the list changes shape .

### Real-World Cases

- **Editable lists:** Stable keys prevent editing state from jumping to the wrong row.
- **Animations:** Keys enable enter/exit animations to target the correct items.
- **Filtered lists:** Stable keys preserve item identity when the filter changes.

### References

- Keys & List Reconciliation — https://raw.githubusercontent.com/ualiyou/frontend-engineering/b5cc4d0dee1575ccd230354e40462e2654dcfc75/docs/02-rendering-frameworks/react/keys-and-list-reconciliation.md#1
- React — Rendering Lists — https://react.dev/learn/rendering-lists

---

## 5. Keys and Identity

### Definitions

**Core Definition:** Keys and identity refer to how React uses the `key` prop to determine whether a child element is the same entity across renders.

**Technical Definition:** Inside a sibling set, React matches new elements to existing fibers by key first, position second. With no key, position is the identity. With a key, the key overrides position: React treats elements with different keys as different identities even at the same position. This is why giving a component a different key resets its state .

**Beginner-Friendly Explanation:** Keys are identity cards. If React sees the same key, it thinks "this is the same item." If it sees a different key, it thinks "this is a different item, throw away the old one and create a new one."

### Purposes

- To give React a stable identity for elements across renders.
- To control whether state is preserved or reset.
- To enable efficient matching of list items.
- To make identity explicit rather than positional.

### Syntax Rules and Structure

| Scenario | Identity |
|----------|----------|
| No key | Position (index) |
| Same key | Same identity; state preserved |
| Different key | Different identity; state reset |
| Same key at same position | Same identity; props updated |
| Same key at different position | Same identity; DOM node moved |

**Syntax Rules:**

1. With no key, position is the identity .
2. With a key, the key overrides position as the identity .
3. Changing a component's key forces a remount and resets state .

### Annotated Code Examples

```jsx
// Same key at different positions: identity is preserved
<ul>
  {items.map(item => <li key={item.id}>{item.text}</li>)}
</ul>
```

**Expected Output:** React moves the existing `li` DOM nodes to their new positions instead of recreating them.

**Why this output:** The key overrides position, so React recognises the items even when they move .

```jsx
// Different key at the same position: identity is reset
<Counter key="a" />
<Counter key="b" />
```

**Expected Output:** The second render destroys the old `Counter` and mounts a new one with fresh state.

**Why this output:** Different keys mean different identities, even at the same position .

### Real-World Cases

- **Resetting form state:** Changing the key of a form component clears its state.
- **Animating list reorders:** Stable keys let React move DOM nodes instead of recreating them.
- **Tab panels:** Different keys per tab reset scroll position and state.

### References

- Keys & List Reconciliation — https://raw.githubusercontent.com/ualiyou/frontend-engineering/b5cc4d0dee1575ccd230354e40462e2654dcfc75/docs/02-rendering-frameworks/react/keys-and-list-reconciliation.md#1
- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state

---

## 6. Keys and State

### Definitions

**Core Definition:** Keys and state refer to how React decides whether to preserve or reset a component's state based on its key and position in the tree.

**Technical Definition:** React keeps state for a component as long as it renders at the same position with the same component type and the same key. Changing the key resets state even at the same position. Changing the component type resets state even at the same position. This behaviour lets developers deliberately reset state by changing a key, or preserve state by keeping the key stable .

**Beginner-Friendly Explanation:** React remembers a component's state only if the component stays the "same component" in the same place. If you change its key, React treats it as a new component and starts fresh. This is useful when you want to reset a form or a widget.

### Purposes

- To understand why state is lost or preserved during reconciliation.
- To deliberately reset state by changing a key.
- To preserve state across list reorders with stable keys.
- To avoid accidental state resets caused by unstable keys.

### Syntax Rules and Structure

| Situation | State |
|-----------|-------|
| Same type, same position, same key | Preserved |
| Same type, same position, different key | Reset |
| Different type, same position | Reset |
| Same key, different position | Preserved (DOM node moved) |

**Syntax Rules:**

1. State is tied to identity (type + key + position) .
2. Changing the key resets state .
3. Changing the component type resets state .
4. Stable keys preserve state across reorders .

### Annotated Code Examples

```jsx
// Resetting a form by changing its key
<Form key={selectedUserId} user={selectedUser} />
```

**Expected Output:** When `selectedUserId` changes, the `Form` remounts with fresh state for the new user.

**Why this output:** Changing the key gives the `Form` a new identity, so React destroys the old instance and mounts a new one .

```jsx
// Preserving state across reorders with stable keys
{todos.map(todo => (
  <EditableTodoItem key={todo.id} todo={todo} />
))}
```

**Expected Output:** Editing state stays with the correct todo when the list is reordered.

**Why this output:** The stable `todo.id` key preserves the identity of each item, so React moves the existing component instances rather than remounting them .

### Real-World Cases

- **User profile forms:** Changing the user key resets the form for the new user.
- **Todo lists:** Stable keys preserve edit state across reorders.
- **Tab panels:** Per-tab keys reset scroll position and internal state.

### References

- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state
- Stack Overflow — Keys and state preservation — https://stackoverflow.com

---

## 7. Keys and Performance

### Definitions

**Core Definition:** Keys and performance refer to how key choice affects the number of DOM operations and re-renders React performs during reconciliation.

**Technical Definition:** Stable, unique keys let React match list items by identity, performing O(n) moves and updates. Index keys cause React to match by position, which means that when the list changes shape, React updates every item's props instead of moving a single item. This leads to unnecessary re-renders, lost DOM state, and performance degradation. Random keys cause full remounts on every render .

**Beginner-Friendly Explanation:** Good keys help React work less. If you use stable IDs, React can move one item when the list changes. If you use index keys, React might think every item changed and redo all of them.

### Purposes

- To minimise DOM operations during list updates.
- To avoid unnecessary re-renders caused by unstable keys.
- To preserve DOM state (focus, input values) and avoid expensive remounts.
- To keep large lists responsive during sorting and filtering.

### Syntax Rules and Structure

| Key type | Performance impact |
|----------|-------------------|
| Stable data ID | O(n) moves/updates; minimal DOM work |
| Index | O(n) prop updates on every shape change; state corruption |
| Random | O(n) full remounts on every render; worst performance |

**Syntax Rules:**

1. Use stable IDs to enable efficient moves .
2. Avoid index keys when the list can change shape .
3. Never use random keys .

### Annotated Code Examples

```jsx
// ✅ Stable keys: one item moves, others are untouched
<ul>
  {items.map(item => (
    <ExpensiveItem key={item.id} item={item} />
  ))}
</ul>
```

**Expected Output:** Inserting an item at the top moves the existing DOM nodes and renders only the new item.

**Why this output:** Stable keys let React match existing items by identity and move them, avoiding prop updates on unchanged items .

```jsx
// ❌ Index keys: every item updates its props on insertion
<ul>
  {items.map((item, index) => (
    <ExpensiveItem key={index} item={item} />
  ))}
</ul>
```

**Expected Output:** Inserting an item at the top causes React to update the props of every existing item.

**Why this output:** Index keys renumber, so React thinks every position now holds a different item .

### Real-World Cases

- **Large data tables:** Stable row keys keep sorting fast.
- **Chat feeds:** Message IDs prevent full re-renders when new messages arrive.
- **Filtered lists:** Stable keys preserve identity when the filter changes.

### References

- Key Stability in Lists — https://stevekinney.com/courses/react-performance/key-stability-in-lists
- CoreUI — How to use the key prop in React lists — https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/

---

## 8. Keys Pitfalls

### Definitions

**Core Definition:** Keys pitfalls are the common mistakes that cause React to misidentify elements, lose state, or perform unnecessary work.

**Technical Definition:** The primary pitfalls are: using array indices as keys (breaks on reorder, insertion, deletion, filtering), using random keys (causes full remounts on every render), using non-unique keys (causes React to mis-match siblings), and using keys that are not derived from data identity. These mistakes lead to state being associated with the wrong item, lost focus and input values, broken animations, and performance degradation .

**Beginner-Friendly Explanation:** These are the traps that make React get confused about which item is which. The biggest one is using the array index as the key — it works until the list changes, and then everything breaks.

### Common Pitfalls Table

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Index as key** | State follows position, not item; breaks on reorder | Use stable data ID  |
| **Random key** | Full remount on every render; destroys state | Never use `Math.random()`  |
| **Non-unique key** | React mis-matches siblings; rendering bugs | Ensure keys are unique among siblings |
| **Key not from data** | Identity does not reflect the actual item | Derive key from item identity  |
| **Changing keys unnecessarily** | Forces remounts and state resets | Keep keys stable across renders |

### Annotated Code Examples

```jsx
// ❌ Index key: state follows position
{todos.map((todo, index) => (
  <EditableTodoItem key={index} todo={todo} />
))}
```

**Expected Output:** Editing “Buy groceries” then inserting an item at the top causes the editing state to jump to the wrong todo.

**Why this output:** Index keys renumber. React thinks position 2 is a different item, so it applies the editing state to the wrong todo .

```jsx
// ❌ Random key: full remount every render
{todos.map(todo => (
  <TodoItem key={Math.random()} todo={todo} />
))}
```

**Expected Output:** Every render destroys and recreates every `TodoItem`, losing all state and causing poor performance.

**Why this output:** Random keys are never stable, so React treats every item as new on every render .

```jsx
// ✅ Stable data ID
{todos.map(todo => (
  <TodoItem key={todo.id} todo={todo} />
))}
```

**Expected Output:** State stays with the correct todo across reorders, insertions, and deletions.

**Why this output:** `todo.id` is stable and unique, so React matches items by identity .

### Real-World Cases

- **Editable lists:** Index keys cause edits to jump to the wrong row.
- **Animated lists:** Random keys break enter/exit animations.
- **Form lists:** Unstable keys lose input focus and values.

### References

- Key Stability in Lists — https://stevekinney.com/courses/react-performance/key-stability-in-lists
- CoreUI — How to use the key prop in React lists — https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/
- DeepSource — Avoid using an element's index as the key prop — https://app.deepsource.com

---

## 9. Element Identity

### Definitions

**Core Definition:** Element identity is what makes React treat two element descriptions as the "same" element across renders.

**Technical Definition:** Element identity is determined by the element's type, its key, and its position in the tree. React uses these signals to decide whether to preserve the existing DOM node and component instance or to tear it down and create a new one. Same type + same key + same position = same identity. Different type, different key, or different position = different identity .

**Beginner-Friendly Explanation:** Element identity is React's answer to the question: "Is this the same thing as before, or something new?" The answer determines whether React updates it or replaces it.

### Purposes

- To understand when React preserves DOM nodes and state.
- To understand when React destroys and recreates elements.
- To control identity through types, keys, and position.

### Syntax Rules and Structure

| Signal | Effect on identity |
|--------|-------------------|
| Type | Different types → different identity; full rebuild  |
| Key | Different keys → different identity; state reset  |
| Position | Without keys, position is the identity  |

### Annotated Code Examples

```jsx
// Same type, same key, same position: identity preserved
<Counter key="a" />
<Counter key="a" />
```

**Expected Output:** The `Counter` instance and its state are preserved.

**Why this output:** All three identity signals (type, key, position) are the same .

```jsx
// Different type at the same position: identity reset
<div><Counter /></div>
<span><Counter /></span>
```

**Expected Output:** The old `Counter` is destroyed and a new one is mounted.

**Why this output:** The root types differ, so React tears down the old tree .

### Real-World Cases

- **Conditional rendering:** Switching types resets state, which can be a bug or a feature.
- **Tab switching:** Different keys per tab reset state.
- **List items:** Stable keys preserve identity across reorders.

### References

- React — Reconciliation — https://ru.react.js.org/docs/reconciliation.html
- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state

---

## 10. Component Identity

### Definitions

**Core Definition:** Component identity is whether React treats a component instance as the same across renders, preserving its state, or as a new instance, resetting its state.

**Technical Definition:** A component instance retains its state as long as it renders at the same position with the same component type and the same key. Changing the key resets state even at the same position. Changing the component type resets state even at the same position. This is why React can keep two `Counter` components side by side with independent state .

**Beginner-Friendly Explanation:** Component identity is whether React sees "the same component" or "a new component." If it is the same, the component keeps its state. If it is new, it starts fresh.

### Purposes

- To understand state preservation and reset behaviour.
- To deliberately reset a component by changing its key.
- To avoid accidental state resets caused by unstable identity.
- To design components that behave predictably across renders.

### Syntax Rules and Structure

| Situation | Component identity |
|-----------|-------------------|
| Same type, same key, same position | Same instance; state preserved |
| Same type, different key | New instance; state reset |
| Different type, same position | New instance; state reset |

**Syntax Rules:**

1. Component identity is determined by type, key, and position .
2. Changing the key forces a new instance .
3. Changing the type forces a new instance .

### Annotated Code Examples

```jsx
function App() {
  const [userId, setUserId] = useState(1);
  return <UserProfile key={userId} userId={userId} />;
}
```

**Expected Output:** Changing `userId` destroys the old `UserProfile` and mounts a new one with fresh state.

**Why this output:** The key changes, so React treats the component as a new identity .

### Real-World Cases

- **User profile forms:** Changing the user key resets the form.
- **Tab content:** Different keys per tab reset scroll and state.
- **Lists:** Stable keys preserve component identity across reorders.

### References

- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state
- Stack Overflow — Component identity and keys — https://stackoverflow.com

---

## 11. Type Identity

### Definitions

**Core Definition:** Type identity is the rule that elements of different types produce different trees and therefore different identities.

**Technical Definition:** React's diffing algorithm assumes that two elements of different types will produce different trees. When the root elements have different types, React tears down the old tree and builds the new tree from scratch. This applies to DOM element types (`<a>` vs `<img>`) and component types (`<Article>` vs `<Comment>`). All state in the old tree is lost .

**Beginner-Friendly Explanation:** Type identity means React looks at the kind of element. A `<div>` is a different kind of thing from a `<span>`. If you swap one for the other, React throws away the old thing and builds a new one.

### Purposes

- To understand why changing element types resets state.
- To deliberately reset a subtree by changing its type.
- To avoid accidental state loss caused by type changes.

### Syntax Rules and Structure

| Change | Result |
|--------|--------|
| `<div>` → `<span>` | Full rebuild; state lost  |
| `<Article>` → `<Comment>` | Full rebuild; state lost  |
| Same type | Node kept; attributes updated  |

**Syntax Rules:**

1. Different types at any level cause a full rebuild of that subtree .
2. Same DOM types keep the DOM node .
3. Same component types keep the instance .

### Annotated Code Examples

```jsx
// Different types: full rebuild
<div><Counter /></div>
<span><Counter /></span>
```

**Expected Output:** The old `Counter` is unmounted and a new one is mounted; state is lost.

**Why this output:** The root types differ, so React tears down the old tree .

```jsx
// Same type: attributes updated, node kept
<div className="before" title="stuff" />
<div className="after" title="stuff" />
```

**Expected Output:** The same DOM node is kept; only `className` is updated.

**Why this output:** Both elements are `div` types, so React updates only the changed attribute .

### Real-World Cases

- **Conditional UI:** Swapping a `<div>` for a `<span>` resets child state.
- **Component refactoring:** Renaming a component resets all instances.
- **Dynamic element types:** Changing the `as` prop of a polymorphic component resets state.

### References

- React — Reconciliation: Elements Of Different Types — https://ru.react.js.org/docs/reconciliation.html

---

## 12. Position Identity

### Definitions

**Core Definition:** Position identity is the rule that, in the absence of keys, React uses an element's position among its siblings as its identity.

**Technical Definition:** By default, when recursing on the children of a DOM node, React iterates over both lists of children at the same time and generates a mutation whenever there is a difference. Without keys, the index (position) becomes the identity. This means inserting an item at the top shifts every element into a different slot, and React updates each fiber with the wrong item's props .

**Beginner-Friendly Explanation:** Position identity means React assumes the first child is the first child, the second is the second, and so on. If you insert something at the top, React thinks every item changed, because the first child is now a different item.

### Purposes

- To understand default reconciliation behaviour without keys.
- To recognise why index-based matching breaks on reorder.
- To appreciate why keys are necessary for dynamic lists.

### Syntax Rules and Structure

| Situation | Identity |
|-----------|----------|
| No key | Position (index)  |
| Key present | Key overrides position  |

**Syntax Rules:**

1. Without keys, position is the identity .
2. Inserting at the top shifts every position .
3. Keys override position to provide stable identity .

### Annotated Code Examples

```jsx
// No keys: position is identity
<ul>
  <li>A</li>
  <li>B</li>
</ul>
// After inserting D at the top:
<ul>
  <li>D</li>
  <li>A</li>
  <li>B</li>
</ul>
```

**Expected Output:** React thinks the first item changed from A to D, the second from B to A, and B is new.

**Why this output:** Without keys, React matches by position. Inserting at the top shifts every item into a new position .

### Real-World Cases

- **Static lists:** Position identity is fine for lists that never change.
- **Dynamic lists:** Position identity causes state corruption on reorder.
- **Append-only lists:** Position identity works because existing positions do not shift.

### References

- React — Reconciliation: Recursing On Children — https://ru.react.js.org/docs/reconciliation.html
- Keys & List Reconciliation — https://raw.githubusercontent.com/ualiyou/frontend-engineering/b5cc4d0dee1575ccd230354e40462e2654dcfc75/docs/02-rendering-frameworks/react/keys-and-list-reconciliation.md#1

---

## 13. Reconciliation Heuristics

### Definitions

**Core Definition:** Reconciliation heuristics are the two assumptions React makes to reduce tree diffing from O(n³) to O(n).

**Technical Definition:** React implements a heuristic O(n) algorithm based on two assumptions: (1) two elements of different types will produce different trees, and (2) the developer can hint at which child elements may be stable across different renders with a `key` prop. These assumptions are valid for almost all practical use cases and enable React to compare trees in linear time .

**Beginner-Friendly Explanation:** Heuristics are educated guesses that make the algorithm fast. React guesses that different types mean different trees, and that keys tell it which children are stable. These guesses are almost always right, so React can be fast without being perfect.

### Purposes

- To understand why React's algorithm is O(n) instead of O(n³).
- To understand the assumptions that make reconciliation fast.
- To recognise when those assumptions might not hold.

### Syntax Rules and Structure

| Assumption | Consequence |
|------------|-------------|
| Different types → different trees | Full rebuild on type change  |
| Keys hint at stable children | O(n) list matching with keys  |

**Syntax Rules:**

1. React does not try to match elements across different tree levels .
2. React assumes different types produce different trees .
3. Keys are the developer's hint for stable identity .

### Annotated Code Examples

```jsx
// Heuristic in action: different types cause full rebuild
<div><Counter /></div>
<span><Counter /></span>
```

**Expected Output:** Full rebuild because the root types differ.

**Why this output:** React's first heuristic assumes different types produce different trees .

```jsx
// Heuristic in action: keys enable O(n) matching
{todos.map(todo => <TodoItem key={todo.id} todo={todo} />)}
```

**Expected Output:** O(n) matching by key.

**Why this output:** React's second heuristic uses keys to match children efficiently .

### Real-World Cases

- **Type changes:** Swapping components resets state, following the heuristic.
- **Keyed lists:** Keys enable efficient updates, following the heuristic.
- **Cross-level moves:** React does not move elements across levels, so moving a component to a different parent remounts it.

### References

- React — Reconciliation: Heuristics — https://ru.react.js.org/docs/reconciliation.html
- React — Reconciliation (Legacy) — https://he.legacy.reactjs.org/docs/reconciliation.html

---

## 14. Reconciliation Performance

### Definitions

**Core Definition:** Reconciliation performance is the cost of comparing element trees and the efficiency with which React applies the resulting DOM changes.

**Technical Definition:** Reconciliation performance depends on tree depth, element count, type stability, and key quality. React's Fiber architecture makes reconciliation incremental and interruptible, splitting work into chunks and prioritising updates. The stack reconciler, used before React 16, was purely recursive and could not be interrupted, so large diffs could block the main thread. Fiber reimagines reconciliation as an incremental process, allowing React to pause, resume, and abort work .

**Beginner-Friendly Explanation:** Reconciliation performance is about how fast React can compare the old and new screens. If your tree is deep or your list is huge, the comparison takes longer. Fiber helps by breaking the work into small pieces so the browser can stay responsive.

### Purposes

- To understand the performance characteristics of reconciliation.
- To identify what makes reconciliation slow (deep trees, large lists, unstable keys).
- To leverage Fiber's interruptible rendering for responsiveness.
- To apply keys and type stability to reduce reconciliation work.

### Syntax Rules and Structure

| Factor | Impact |
|--------|--------|
| Tree depth | More elements to compare |
| List size | More children to match |
| Type instability | Full rebuilds on type changes |
| Key quality | Index keys cause unnecessary prop updates; stable keys enable moves |
| Fiber scheduling | Interruptible work keeps the main thread responsive |

**Syntax Rules:**

1. Fiber reconciliation is interruptible; the stack reconciler was not .
2. Stable keys reduce the number of DOM operations .
3. Type stability avoids full rebuilds .

### Annotated Code Examples

```jsx
// Stable keys + stable types: efficient reconciliation
<ul>
  {items.map(item => (
    <ExpensiveItem key={item.id} item={item} />
  ))}
</ul>
```

**Expected Output:** React moves existing DOM nodes and renders only the new item.

**Why this output:** Stable keys and types let React match by identity and avoid unnecessary work .

### Real-World Cases

- **Large lists:** Stable keys and virtualization keep reconciliation fast.
- **Deep trees:** Flattening the tree reduces reconciliation depth.
- **Frequent updates:** Fiber's interruptible rendering keeps the UI responsive.

### References

- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/
- Virtual DOM & Fiber Architecture — https://stevekinney.com/courses/react-performance/virtual-dom-fiber-architecture

---

## 15. Reconciliation Best Practices

### Definitions

**Core Definition:** Reconciliation best practices are the guidelines for structuring elements, types, and keys to make React's diffing efficient and predictable.

**Technical Definition:** Best practices centre on: using stable, unique data IDs for keys; avoiding index and random keys; keeping element types stable where state should be preserved; deliberately changing keys or types to reset state; keeping tree depth reasonable; and recognising that Fiber's interruptible rendering depends on pure components and stable identity. A correct key is stable, unique among its siblings, and derived from the data's identity — usually a server ID .

**Beginner-Friendly Explanation:** Use real IDs for keys, not indices. Keep component types the same when you want to keep state. Change the key when you want to reset state. Keep your trees reasonably flat. These habits keep React fast and your UI predictable.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use stable data IDs** | Key from `item.id`, not index  |
| **Never use random keys** | They cause full remounts  |
| **Avoid index keys for dynamic lists** | Index keys break on reorder/insert/delete  |
| **Keep types stable to preserve state** | Same type preserves instance  |
| **Change keys to reset state deliberately** | Different key resets instance  |
| **Keep trees reasonably flat** | Less depth means less reconciliation work |
| **Recognise cross-level moves remount** | React does not move elements across levels  |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// Stable keys, stable types, deliberate resets
<ul>
  {todos.map(todo => (
    <TodoItem key={todo.id} todo={todo} />
  ))}
</ul>

<Form key={selectedUserId} user={selectedUser} />
```

**❌ Bad Practices:**

```jsx
// Index keys, random keys, unnecessary remounts
{todos.map((todo, index) => (
  <TodoItem key={index} todo={todo} />
))}

{todos.map(todo => (
  <TodoItem key={Math.random()} todo={todo} />
))}
```

**Expected Output:** The good practices produce predictable state and efficient DOM updates. The bad practices cause state corruption and unnecessary work.

**Why this output:** Stable keys preserve identity; deliberate key changes reset state. Index keys break on list shape changes; random keys cause full remounts .

### Real-World Cases

- **All list rendering:** Stable keys are the baseline best practice.
- **Form resets:** Changing the key is the idiomatic way to reset a form.
- **Performance-sensitive lists:** Stable keys and flat trees minimise reconciliation cost.

### References

- Keys & List Reconciliation — https://raw.githubusercontent.com/ualiyou/frontend-engineering/b5cc4d0dee1575ccd230354e40462e2654dcfc75/docs/02-rendering-frameworks/react/keys-and-list-reconciliation.md#1
- Key Stability in Lists — https://stevekinney.com/courses/react-performance/key-stability-in-lists
- CoreUI — How to use the key prop in React lists — https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/
- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state

---

## References

- React — Reconciliation — https://ru.react.js.org/docs/reconciliation.html
- React — Reconciliation (Legacy) — https://he.legacy.reactjs.org/docs/reconciliation.html
- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state
- React — Rendering Lists — https://react.dev/learn/rendering-lists
- Keys & List Reconciliation — https://raw.githubusercontent.com/ualiyou/frontend-engineering/b5cc4d0dee1575ccd230354e40462e2654dcfc75/docs/02-rendering-frameworks/react/keys-and-list-reconciliation.md#1
- Key Stability in Lists — https://stevekinney.com/courses/react-performance/key-stability-in-lists
- CoreUI — How to use the key prop in React lists — https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/
- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/
- Virtual DOM & Fiber Architecture — https://stevekinney.com/courses/react-performance/virtual-dom-fiber-architecture
- DeepSource — Avoid using an element's index as the key prop — https://app.deepsource.com
- Stack Overflow — Keys and state preservation — https://stackoverflow.com