# React Rendering Lists — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

Rendering lists in React is the process of transforming arrays of data into arrays of JSX elements using JavaScript's array methods, primarily `.map()`, to generate multiple similar components dynamically.

**Technical Definition**

React does not have a dedicated list-rendering syntax. Instead, developers use standard JavaScript array methods—`map()` to transform each data item into a React element, and `filter()` to select subsets—then embed the resulting array of elements inside JSX curly braces. React renders the array by iterating over it and creating a DOM node (or component instance) for each element. The `key` prop provides a stable identity for each element, enabling React's reconciliation algorithm to efficiently update lists when data changes.

**Beginner-Friendly Explanation**

Rendering lists is how you turn a bunch of data into a bunch of UI elements. If you have an array of names, you use `.map()` to turn each name into a list item. React then displays them all. The `key` prop is like a name tag for each item, so React can keep track of which item is which when the list changes.

### Key Characteristics

- **JavaScript-Native**: React uses standard array methods (`map`, `filter`) rather than a template language.
- **Declarative**: You describe what each item should look like, and React handles the rendering.
- **Key-Dependent**: Every element in a mapped array must have a unique `key` prop for correct reconciliation.
- **Efficient**: With stable keys, React can update only the items that changed, preserving state and performance.

### Prerequisites

- JavaScript arrays and array methods (`map`, `filter`)
- JSX syntax and expressions
- React components and props
- Basic understanding of reconciliation

### Related Programming Areas

- **Reconciliation**: React's diffing algorithm that uses keys to match elements.
- **State Management**: Lists often interact with state for adding, removing, or reordering items.
- **Component Composition**: List items are typically rendered as separate components.

### Core Concepts / Features

- Arrays and `.map()`
- Dynamic Component Generation
- `key` Props
- Stable Keys
- Why Keys Matter
- Avoiding Inappropriate Array Indexes as Keys
- Nested Collections

---

## 1. Arrays and `.map()`

### Definitions

**Core Definition**

`.map()` is a JavaScript array method that creates a new array by applying a transformation function to each element of the original array.

**Technical Definition**

`Array.prototype.map(callback)` iterates over each element in an array, calls the `callback` function with the element (and optionally its index), and collects the return values into a new array. In React, the callback returns a JSX element for each data item, producing an array of React elements that can be embedded directly in JSX.

**Beginner-Friendly Explanation**

`.map()` is like a factory assembly line. You give it a list of raw materials (data), and a machine (the callback function) transforms each one into a finished product (a JSX element). The result is a new list of finished products.

### Purposes

- To transform an array of data into an array of JSX elements.
- To render a dynamic number of similar components based on data length.
- To keep rendering logic declarative and data-driven.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const listItems = array.map(item => <li>{item}</li>);
return <ul>{listItems}</ul>;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `array` | The data array to iterate over. |
| `.map()` | JavaScript array method. |
| `item => <li>{item}</li>` | Callback returning a JSX element for each item. |
| `listItems` | The resulting array of JSX elements. |
| `{listItems}` | Embedding the array in JSX. |

**Syntax Rules**

1. **Return a single element per item**: The callback must return one JSX element per array element.
2. **Implicit return works for single-line arrows**: `item => <li>{item}</li>` returns the JSX implicitly.
3. **Explicit return needed for multi-line**: Use `{ return <li>...</li> }` with curly braces for multi-line callbacks.
4. **Embed in curly braces**: The resulting array is embedded in JSX using `{}`.

**Constraints and Limitations**

- The callback must return a React element, not arbitrary values.
- React will warn if elements lack keys (see Key Props section).

### Annotated Complete Code Examples

**Example 1: Basic List Rendering**

```jsx
const people = [
  'Creola Katherine Johnson: mathematician',
  'Mario José Molina-Pasquel Henríquez: chemist',
  'Mohammad Abdus Salam: physicist',
];

export default function List() {
  // Transform each string into a <li> element
  const listItems = people.map(person => <li>{person}</li>);

  return <ul>{listItems}</ul>;
}
```

**Expected Output**

```
• Creola Katherine Johnson: mathematician
• Mario José Molina-Pasquel Henríquez: chemist
• Mohammad Abdus Salam: physicist
```

**Why This Output Occurs**

`people.map()` iterates over each string and returns a `<li>` element containing that string. The resulting array `[<li>...</li>, <li>...</li>, <li>...</li>]` is embedded in `<ul>`. React renders each `<li>` as a DOM node.

**Example 2: Filtering and Mapping**

```jsx
const people = [
  { id: 0, name: 'Creola Katherine Johnson', profession: 'mathematician' },
  { id: 1, name: 'Mario José Molina-Pasquel Henríquez', profession: 'chemist' },
  { id: 2, name: 'Mohammad Abdus Salam', profession: 'physicist' },
];

export default function List() {
  // First filter to chemists, then map to JSX
  const chemists = people.filter(person => person.profession === 'chemist');
  const listItems = chemists.map(person => <li>{person.name}</li>);

  return <ul>{listItems}</ul>;
}
```

**Expected Output**

```
• Mario José Molina-Pasquel Henríquez
```

**Why This Output Occurs**

`filter()` creates a new array containing only the chemist. `map()` transforms that single-element array into a `<li>`. React renders the filtered list, demonstrating how `filter()` and `map()` compose for data-driven rendering.

### Real-World Cases

**Case: Comment List**

A blog post renders comments from an array of comment objects, using `.map()` to create a `Comment` component for each. The number of comments dynamically determines the UI.

### References

- React — Rendering Lists (Arrays and map) - https://18.react.dev/learn/rendering-lists
- React Legacy — Lists and Keys - https://vi.legacy.reactjs.org/docs/lists-and-keys.html

---

## 2. Dynamic Component Generation

### Definitions

**Core Definition**

Dynamic component generation is the technique of rendering a variable number of component instances based on the length of a data array.

**Technical Definition**

When React encounters an array of elements in JSX, it renders each element as a child in the DOM. By generating this array dynamically with `.map()`, developers can render a UI whose structure and number of child components is determined at runtime by the data, rather than being hardcoded in JSX.

**Beginner-Friendly Explanation**

Dynamic component generation means your UI can adapt to the data. If you have 3 items, you get 3 components. If you have 100 items, you get 100 components. You don't have to write out each one manually.

### Purposes

- To render lists whose length is unknown at development time.
- To avoid repetitive hardcoded JSX for similar items.
- To keep UI structure synchronized with data.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{data.map(item => <Component key={item.id} {...item} />)}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `data.map(...)` | Generates the component array. |
| `<Component />` | The component to render for each item. |
| `key={item.id}` | Unique identifier for reconciliation. |
| `{...item}` | Spread operator passing all item properties as props. |

**Syntax Rules**

1. **Use a component for complex items**: For non-trivial list items, create a dedicated component.
2. **Pass data as props**: Spread the item or pass specific fields.
3. **Always include `key`**: See the next section for details.

### Annotated Complete Code Examples

**Example 1: Dynamic User Cards**

```jsx
function UserCard({ user }) {
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}

function UserList({ users }) {
  return (
    <div className="user-list">
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

**Explanation**

`UserList` receives an array of user objects. `users.map()` generates a `UserCard` for each. The number of `UserCard` components is determined by the array length. Each `UserCard` receives its specific user data via the `user` prop.

### Real-World Cases

**Case: Product Grid**

An e-commerce category page renders a grid of product cards from an API response. The number of products varies by category, and the grid adapts automatically.

### References

- React — Rendering Lists (Multiple Components) - https://18.react.dev/learn/rendering-lists

---

## 3. `key` Props

### Definitions

**Core Definition**

The `key` prop is a special string attribute that React uses to uniquely identify each element in a list, enabling efficient reconciliation across renders.

**Technical Definition**

`key` is a reserved prop that React extracts from the element's props and uses internally to match elements between the previous and new render trees. Keys must be unique among sibling elements (elements at the same level in the same array) but do not need to be globally unique.

**Beginner-Friendly Explanation**

The `key` prop is like a name tag for each list item. When React re-renders the list, it looks at the name tags to figure out which items are the same, which are new, and which were removed.

### Purposes

- To give each list item a stable identity across renders.
- To enable React to match elements between renders.
- To preserve component state in list items when the list changes.
- To optimize rendering performance by minimizing DOM operations.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{items.map(item => <Item key={item.id} data={item} />)}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `key={item.id}` | Unique, stable identifier from data. |
| `item.id` | Typically the database ID or UUID. |

**Syntax Rules**

1. **Keys must be unique among siblings**: Two elements in the same array cannot have the same key.
2. **Keys don't need to be globally unique**: The same key can appear in different arrays.
3. **Place key on the array element, not inside the component**: The `key` belongs on the `<Item key={...} />` element in the `.map()` call, not on the root element inside the `Item` component.
4. **Keys are not accessible via props**: React extracts `key` and does not pass it to the component's props.

**Constraints and Limitations**

- Using unstable keys (e.g., `Math.random()`) causes all elements to be recreated every render.
- Missing keys cause a console warning and degrade performance.

### Annotated Complete Code Examples

**Example 1: Correct Key Placement**

```jsx
// ✅ Correct: key on the <Item /> in the map call
function List({ items }) {
  return (
    <ul>
      {items.map(item => (
        <Item key={item.id} value={item.value} />
      ))}
    </ul>
  );
}

// ✅ Correct: Item component does NOT need key internally
function Item({ value }) {
  return <li>{value}</li>;
}
```

**Explanation**

The `key` is placed on the `<Item />` element inside the `.map()` callback. This is where React looks for keys during reconciliation. The `Item` component itself does not need to know about the key.

### Real-World Cases

**Case: Chat Message List**

Messages use `key={message.id}` to preserve scroll position and input state when new messages arrive. Without stable keys, the input field would lose focus.

### References

- React Legacy — Keys - https://vi.legacy.reactjs.org/docs/lists-and-keys.html
- CoreUI — How to use the key prop in React lists - https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/

---

## 4. Stable Keys

### Definitions

**Core Definition**

Stable keys are key values that remain consistent for the same logical item across renders, regardless of the item's position in the list.

**Technical Definition**

A stable key is a value derived from the item's inherent identity (e.g., a database ID, UUID, or unique property) rather than its position or a value that changes between renders. Stability ensures React can recognize the same item across re-renders, reorders, and data updates.

**Beginner-Friendly Explanation**

A stable key is like a person's social security number. It doesn't change when they move to a different position in a line. React uses this permanent identifier to keep track of who's who.

### Purposes

- To preserve component state across list updates.
- To enable React to correctly identify moved items.
- To prevent unnecessary re-renders and remounts.
- To ensure correct behavior for animations and focus.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{items.map(item => <Item key={item.id} />)}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `item.id` | A stable, unique property from the data. |
| `item.uuid` | Alternative stable identifier. |
| Composite key | `key={`${item.date}-${item.userId}`}` when no single ID exists. |

**Syntax Rules**

1. **Derive from data**: Use IDs, UUIDs, or unique fields from the data source.
2. **Composite keys**: When no single unique field exists, combine fields: `key={`${item.category}-${item.name}`}`.
3. **Generate IDs when creating items**: If data lacks IDs, generate them when the item is first created (e.g., `uuidv4()`), not during render.

**Constraints and Limitations**

- Keys must be unique among siblings at each render.
- Keys should be strings or numbers; other types are coerced to strings.

### Annotated Complete Code Examples

**Example 1: Stable ID from Data**

```jsx
const todos = [
  { id: 't1', text: 'Buy groceries' },
  { id: 't2', text: 'Walk the dog' },
];

function TodoList() {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

**Explanation**

Each todo has a unique `id`. Using `key={todo.id}` ensures that even if the todos are reordered or filtered, React recognizes each item by its ID, not its position.

**Example 2: Composite Key**

```jsx
const events = [
  { date: '2025-01-15', userId: 'u1', title: 'Meeting' },
  { date: '2025-01-15', userId: 'u2', title: 'Lunch' },
];

function EventList() {
  return (
    <ul>
      {events.map(event => (
        <li key={`${event.date}-${event.userId}`}>{event.title}</li>
      ))}
    </ul>
  );
}
```

**Explanation**

No single field is unique, but the combination of `date` and `userId` is. The composite key provides a stable identity for each event.

### Real-World Cases

**Case: Database-Driven Lists**

Any list rendered from a database uses the record's primary key as the `key` prop, ensuring stability across data refetches.

### References

- Steve Kinney — Key Stability in Lists - https://stevekinney.com/courses/react-performance/key-stability-in-lists
- React Doctor — no-array-index-as-key - https://mintlify.wiki/millionco/react-doctor/rules/correctness

---

## 5. Why Keys Matter

### Definitions

**Core Definition**

Keys matter because they are the mechanism React uses to match elements in a list between renders, determining which DOM nodes and component instances to keep, update, move, or destroy.

**Technical Definition**

During reconciliation, React compares the previous element tree with the new one. For lists, React uses the `key` prop to match old elements with new ones. Without keys (or with unstable keys), React falls back to matching by position, which causes incorrect element matching when the list order changes. Incorrect matching leads to state loss, unnecessary DOM operations, and visual bugs.

**Beginner-Friendly Explanation**

Keys matter because without them, React gets confused about which item is which. Imagine a class photo where everyone moves around. Without name tags, you can't tell who's who. With name tags (keys), you can track each person even if they move.

### Purposes

- To ensure React updates the correct DOM nodes.
- To preserve component state in the right items.
- To optimize performance by minimizing DOM mutations.
- To enable correct animations and focus management.

### Syntax Rules and Structure

**Conceptual Comparison**

```
With stable keys:
Old: [A(id=1), B(id=2), C(id=3)]
New: [D(id=4), A(id=1), B(id=2), C(id=3)]
React: "D is new, A/B/C moved" → Insert D, move A/B/C

With index keys:
Old: [A(0), B(1), C(2)]
New: [D(0), A(1), B(2), C(3)]
React: "Position 0 changed A→D, 1 changed B→A, 2 changed C→B, 3 is new"
→ Updates every item, loses state
```

**Syntax Rules**

1. **Keys are required for dynamic lists**: Any list that can reorder, filter, or change length needs stable keys.
2. **Keys are optional for static lists**: A hardcoded list that never changes may omit keys without issues (though React may still warn).

### Annotated Complete Code Examples

**Example 1: State Loss with Index Keys**

```jsx
// ❌ Bug: input state moves to wrong item on reorder
function BuggyList({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>
          <input placeholder={item.name} />
        </li>
      ))}
    </ul>
  );
}
```

**Scenario**

1. Items: `[{name: 'A'}, {name: 'B'}, {name: 'C'}]` — type "hello" next to A.
2. Reorder to: `[{name: 'C'}, {name: 'A'}, {name: 'B'}]`.
3. The "hello" text stays at position 0, now next to C — wrong item!

**Why This Happens**

With `key={index}`, React matches by position. Position 0 was A, now is C. React reuses the component instance at position 0 (which has the "hello" state) and just changes its props to C. The state stays with the position, not the item.

**Example 2: Correct State Preservation**

```jsx
// ✅ Fixed: input state follows the item
function FixedList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <input placeholder={item.name} />
        </li>
      ))}
    </ul>
  );
}
```

**Explanation**

With `key={item.id}`, React recognizes that A moved from position 0 to position 1. The component instance (and its "hello" state) moves with it. The state stays with the correct item.

### Real-World Cases

**Case: Editable Tables**

An editable data table with inline editing uses stable keys. When rows are sorted or filtered, the editing state stays with the correct row, not the position.

### References

- GitHub — list-key-reconciliation (Interactive Demo) - https://github.com/dev48v/list-key-reconciliation
- Steve Kinney — Key Stability in Lists - https://stevekinney.com/courses/react-performance/key-stability-in-lists

---

## 6. Avoiding Inappropriate Array Indexes as Keys

### Definitions

**Core Definition**

Using array index as a key is inappropriate for dynamic lists because indices change when items are added, removed, or reordered, causing React to match elements incorrectly.

**Technical Definition**

When `key={index}` is used, React associates the component instance with the item's position rather than the item's identity. If the list order changes, the component at position 0 is reused for a different data item, leading to state being attached to the wrong item. This causes bugs in inputs, focus, animations, and any component with internal state.

**Beginner-Friendly Explanation**

Using the index as a key is like labeling students by their seat number instead of their name. When someone moves seats, you think a different person is sitting there. The index doesn't follow the item.

### Purposes (of Avoiding)

- To prevent state from being attached to the wrong item.
- To avoid unnecessary re-renders and remounts.
- To ensure animations and focus behave correctly.
- To prevent data corruption in forms.

### Syntax Rules and Structure

**Problematic Pattern**

```jsx
// ❌ Avoid for dynamic lists
{items.map((item, index) => <Item key={index} />)}
```

**Correct Pattern**

```jsx
// ✅ Prefer stable IDs
{items.map(item => <Item key={item.id} />)}
```

**When Index is Acceptable**

```jsx
// ✅ OK for static lists that never reorder
const navLinks = ['Home', 'About', 'Contact'];
{navLinks.map((link, index) => <li key={index}>{link}</li>)}

// ✅ OK for placeholder skeletons
{Array.from({ length: 5 }).map((_, i) => <Skeleton key={i} />)}
```

**Syntax Rules**

1. **Index is acceptable when**: The list is static (never reorders), items are never inserted/removed except at the end, and items have no internal state.
2. **Index is inappropriate when**: The list can be sorted, filtered, reordered, or items can be added/removed in the middle.
3. **Default to stable IDs**: When in doubt, use a stable identifier from the data.

### Annotated Complete Code Examples

**Example 1: The Index Key Bug**

```jsx
function ReorderableList() {
  const [items, setItems] = useState([
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
  ]);

  function moveBobUp() {
    setItems([items[1], items[0]]); // Swap order
  }

  return (
    <div>
      <button onClick={moveBobUp}>Move Bob Up</button>
      {items.map((item, index) => (
        <div key={index}>
          <input placeholder={item.name} />
        </div>
      ))}
    </div>
  );
}
```

**Scenario**

1. Type "hello" in Alice's input (position 0).
2. Click "Move Bob Up."
3. The "hello" text moves to Bob's row, even though it belonged to Alice.

**Why This Happens**

With `key={index}`, React reuses the component at position 0 for Bob. The input state ("hello") stays with the position, not the item.

### Real-World Cases

**Case: Todo List with Drag-and-Drop**

A drag-and-drop todo list uses stable IDs as keys. When items are dragged to new positions, the input values and checkbox states follow the correct items.

### References

- React Doctor — no-array-index-as-key Rule - https://mintlify.wiki/millionco/react-doctor/rules/correctness
- GitHub — list-key-reconciliation (Why index keys break state) - https://github.com/dev48v/list-key-reconciliation

---

## 7. Nested Collections

### Definitions

**Core Definition**

Nested collections are lists within lists, where each top-level item contains its own array of sub-items that must also be rendered with `.map()`.

**Technical Definition**

When data has a hierarchical structure (e.g., categories containing products, or folders containing files), rendering requires nested `.map()` calls. Each level of nesting needs its own `key` props, and keys only need to be unique among siblings at the same level, not globally.

**Beginner-Friendly Explanation**

Nested collections are like a menu with categories and items. Each category has its own list of dishes. You render the categories, and inside each category, you render its dishes.

### Purposes

- To render hierarchical data structures.
- To display tree views, menus, or nested comments.
- To handle data with parent-child relationships.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{categories.map(category => (
  <div key={category.id}>
    <h2>{category.name}</h2>
    <ul>
      {category.items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  </div>
))}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `categories.map` | Outer list iteration. |
| `key={category.id}` | Key for outer item. |
| `category.items.map` | Inner list iteration. |
| `key={item.id}` | Key for inner item. |

**Syntax Rules**

1. **Each level needs its own keys**: Outer keys for categories, inner keys for items.
2. **Keys unique within siblings only**: An item with `key={1}` in Category A and `key={1}` in Category B is fine.
3. **Nesting depth**: Keep nesting to 2-3 levels for readability; extract components for deeper nesting.

**Constraints and Limitations**

- Deep nesting can harm readability; consider extracting components.
- Each level of nesting adds computational cost during reconciliation.

### Annotated Complete Code Examples

**Example 1: Category and Item Rendering**

```jsx
function CategoryList({ categories }) {
  return (
    <div>
      {categories.map(category => (
        <div key={category.id} className="category">
          <h2>{category.name}</h2>
          <ul>
            {category.items.map(item => (
              <li key={item.id}>{item.name}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

**Expected Output**

```
[Category: Fruits]
• Apple
• Banana

[Category: Vegetables]
• Carrot
• Broccoli
```

**Why This Output Occurs**

The outer `.map()` iterates over categories, rendering each category's heading and its items list. The inner `.map()` iterates over the category's items. Each level has its own keys.

**Example 2: Extracting Nested Components**

```jsx
function Category({ category }) {
  return (
    <div className="category">
      <h2>{category.name}</h2>
      <ul>
        {category.items.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}

function CategoryList({ categories }) {
  return (
    <div>
      {categories.map(category => (
        <Category key={category.id} category={category} />
      ))}
    </div>
  );
}
```

**Explanation**

Extracting the `Category` component keeps the outer `.map()` clean. The `key` for the outer level is on `<Category />`, and the inner level has its own keys on `<li>` elements.

### Real-World Cases

**Case: File Explorer**

A file explorer renders folders (top level) containing files and subfolders (nested levels). Each level uses stable IDs from the file system.

### References

- Steve Kinney — Nested Lists (Key Stability) - https://stevekinney.com/courses/react-performance/key-stability-in-lists
- Tencent Cloud — Rendering Nested Arrays in React - https://cloud.tencent.cn/developer/information/如何在React中呈现对象中的嵌套数组？

---

## References

- React Official Documentation — Rendering Lists - https://18.react.dev/learn/rendering-lists
- React Legacy Documentation — Lists and Keys - https://vi.legacy.reactjs.org/docs/lists-and-keys.html
- React Official Documentation (Chinese) — 渲染列表 - https://zh-hans.react.dev/learn/rendering-lists
- React Official Documentation (Ukrainian) — Рендеринг списків - https://uk.react.dev/learn/rendering-lists
- React Official Documentation (Portuguese) — Renderizando Listas - https://pt-br.react.dev/learn/rendering-lists
- React Legacy Documentation (Chinese Traditional) — 列表與 Key - https://zh-hant.legacy.reactjs.org/docs/lists-and-keys.html
- CoreUI — How to use the key prop in React lists - https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/
- Steve Kinney — Key Stability in Lists - https://stevekinney.com/courses/react-performance/key-stability-in-lists
- GitHub — list-key-reconciliation (Interactive Demo) - https://github.com/dev48v/list-key-reconciliation
- GitHub — CodeYourFuture Curriculum (Keys) - https://raw.githubusercontent.com/CodeYourFuture/curriculum/7ffcf8478ed3d870fd878f68ebaf78c4ff8ce92b/common-content/en/module/react/keys/index.md
- React Doctor — no-array-index-as-key Rule - https://mintlify.wiki/millionco/react-doctor/rules/correctness
- GitHub — ravnhq/ai-toolkit (Stable Keys) - https://github.com/ravnhq/ai-toolkit/blob/main/skills/frontend/tech-react/rules/perf-stable-keys.md
- Tencent Cloud — Rendering Nested Arrays in React - https://cloud.tencent.cn/developer/information/如何在React中呈现对象中的嵌套数组？
- GitHub — react-elements-renderer (Nested Data) - https://github.com/domeafavour/react-elements-renderer
- Syncfusion — Nested List in React ListView - https://ej2.syncfusion.com/react/documentation/listview/nested-list
- OneCompiler — Keys in React - https://onecompiler.com/tutorials/react/lists-and-keys/keys