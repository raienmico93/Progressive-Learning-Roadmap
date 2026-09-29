# Large-Application Performance: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Large-application performance is the discipline of keeping React applications fast as they grow in size, complexity, and data volume—using windowing, state partitioning, concurrent rendering, network optimisation, and bundle engineering to maintain responsiveness regardless of scale.

**Technical Definition:** Large-application performance addresses the compounding performance problems that emerge when a React application scales: rendering thousands of DOM nodes (solved by windowing/virtualization), global state updates that trigger wide re-render cascades (solved by state partitioning and colocation), expensive synchronous updates that block the main thread (solved by concurrent transitions via `useTransition` and `useDeferredValue`), network waterfalls and payload bloat (solved by debouncing, throttling, streaming HTTP, data-fetching caches, and GraphQL payload minimisation), and unbounded JavaScript bundles (solved by bundle visualisation, tree-shaking, and asset/font optimisation). Unlike micro-optimisations (memoising a single component), large-application performance is architectural: it requires deliberate decisions about data flow, rendering strategy, network topology, and delivery pipeline.

**Beginner-Friendly Explanation:** A small app can get away with almost anything—render everything, put all state in one place, load all code upfront. But as the app grows, those shortcuts become bottlenecks. Large-application performance is about fixing the bottlenecks that only appear at scale: rendering a list of 10,000 items, keeping typing responsive while a heavy update runs, avoiding a 5MB JavaScript bundle, and not overwhelming the server with a request per keystroke. This cheat sheet covers the five pillars that make large React apps fast.

### Key Characteristics

- **Architectural, Not Incidental:** Large-app performance is about structure—where state lives, how data flows, what renders when—not micro-tuning individual components.
- **Windowing Beats Optimisation:** Rendering 10,000 DOM nodes is slow no matter how well memoised; windowing renders only the visible ~30.
- **Colocation Over Global State:** The more state lives in a global store, the wider the blast radius of every update.
- **Concurrency Keeps the UI Alive:** Transitions and deferred values let React keep the main thread responsive during heavy renders.
- **Network is Often the Bottleneck:** Debouncing, caching, streaming, and payload minimisation frequently outperform any client-side optimisation.
- **Bundle Size Drives Load Time:** A 5MB bundle is slow on any device; tree-shaking, code splitting, and asset optimisation are non-negotiable.
- **Measurement-Driven:** Every optimisation must be validated by profiling (React DevTools, Lighthouse, WebPageTest, bundle analysers).

### Prerequisites

- Solid understanding of React function components, JSX, and Hooks.
- Working knowledge of rendering performance (re-render triggers, memoisation, referential equality).
- Familiarity with code splitting (`React.lazy`, `<Suspense>`, dynamic `import()`).
- Basic understanding of server-state management (TanStack Query, SWR) and HTTP caching.
- Awareness of bundlers (Vite, Webpack, Turbopack) and build tooling.

### Related Programming Areas

- **Rendering Performance:** Re-render triggers, memoisation, and component boundaries.
- **State Architecture:** Colocation, partitioning, and external stores.
- **Concurrent Rendering:** `useTransition`, `useDeferredValue`, Suspense.
- **Network Optimisation:** Debouncing, throttling, caching, streaming, GraphQL.
- **Build Engineering:** Tree-shaking, bundle analysis, code splitting, asset optimisation.

### Core Concepts / Features

1. Windowing & Virtualization
2. State Partitioning & Colocation
3. Concurrent Transitions
4. Asynchronous & Network Strategy
5. Asset & Bundle Engineering

---

## Core Concept 1: Windowing & Virtualization

### Definitions

**Core Definition:** Windowing (also called virtualization) is the technique of rendering only the DOM nodes currently visible in the viewport, recycling them as the user scrolls, so that a list of 10,000 items renders as smoothly as a list of 30.

**Technical Definition:** Windowing libraries (`react-window`, `react-virtualized`, `@tanstack/react-virtual`) measure the scroll container's viewport and the total content height, calculate which items are visible (plus an overscan buffer), and render only those items. As the user scrolls, the library updates the visible range and reuses the same DOM nodes for new items (item recycling). Variable-height items require dynamic measurement (`ResizeObserver` or `estimateSize`). Windowing reduces DOM node count from O(n) to O(visible), eliminates layout thrashing, and keeps memory usage constant regardless of list size. The trade-off is complexity: windowing libraries require a fixed-height container, careful handling of dynamic content, and manual support for features like sticky headers, grid layouts, and infinite scroll.

**Beginner-Friendly Explanation:** Imagine a bookshelf with 10,000 books. Without windowing, you build all 10,000 books every time someone looks at the shelf. With windowing, you build only the 10 books visible on the current shelf, and when the user scrolls, you swap out the books for new ones. The user sees a smooth scrolling experience, but you only ever built 10 books. That is windowing: render only what is visible.

### Purposes

- To render massive lists (10,000+ items) without freezing the browser.
- To keep DOM node count constant regardless of list size.
- To reduce memory usage and garbage collection pressure.
- To maintain smooth scroll performance at 60fps.
- To support infinite scroll, grids, and variable-height content.

### Syntax Rules and Structure

**react-window (Fixed Size):**
```jsx
import { FixedSizeList } from 'react-window';

const Row = ({ index, style }) => (
  <div style={style}>Row {index}</div>
);

function BigList({ items }) {
  return (
    <FixedSizeList
      height={600}
      width="100%"
      itemCount={items.length}
      itemSize={35}
      overscanCount={5}
    >
      {Row}
    </FixedSizeList>
  );
}
```

**Component Breakdown:**
- `height` / `width`: The scroll container's dimensions.
- `itemCount`: Total number of items.
- `itemSize`: Fixed height (in pixels) of each item.
- `overscanCount`: Number of items rendered outside the viewport (buffer).
- `Row`: Receives `index` and `style` (the `style` must be applied to the row's root element).

**react-window (Variable Size):**
```jsx
import { VariableSizeList } from 'react-window';

function VariableList({ items }) {
  const getItemSize = (index) => items[index].height;

  return (
    <VariableSizeList
      height={600}
      width="100%"
      itemCount={items.length}
      itemSize={getItemSize}
      estimatedItemSize={50}
    >
      {({ index, style }) => (
        <div style={style}>{items[index].content}</div>
      )}
    </VariableSizeList>
  );
}
```

**Component Breakdown:**
- `itemSize` as a function: Returns the height for each index.
- `estimatedItemSize`: Fallback height for items not yet measured.

**TanStack Virtual (Modern, Headless):**
```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
    overscan: 5,
  });

  return (
    <div ref={parentRef} style={{ height: 600, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: virtualItem.size,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            {items[virtualItem.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

**Component Breakdown:**
- `useVirtualizer`: Returns a virtualizer with `getTotalSize()`, `getVirtualItems()`, and `measureElement()`.
- `parentRef`: The scroll container.
- `estimateSize`: Fallback height for each item.
- `transform: translateY(...)`: Positions each item absolutely.
- `measureElement`: Attach to items for dynamic measurement.

**Dynamic Measurement (TanStack Virtual):**
```jsx
function DynamicList({ items }) {
  const parentRef = useRef(null);
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: 600, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            ref={virtualizer.measureElement}
            data-index={virtualItem.index}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            {items[virtualItem.index].content}
          </div>
        ))}
      </div>
    </div>
  );
}
```

**Component Breakdown:**
- `ref={virtualizer.measureElement}`: Measures the item's actual height.
- `data-index`: Required for `measureElement` to map the DOM node to its index.

**Syntax Rules:**
- The scroll container must have a fixed height (or a height determined by a parent).
- Apply the `style` prop from the library to the row's root element.
- Use `overscan` (or `overscanCount`) to render a buffer of items outside the viewport.
- For variable heights, use `measureElement` (TanStack Virtual) or `itemSize` as a function (react-window).
- Avoid expensive operations inside the row renderer; it runs frequently.
- Memoise the row renderer if it receives complex props.
- Test with the actual data volume; windowing benefits only appear at scale.

**Constraints and Limitations:**
- Windowing requires a fixed-height container; layouts that rely on content height need adjustment.
- Dynamic heights require measurement, which adds a render pass.
- Accessibility: screen readers may not see off-screen items; provide alternative navigation (search, page jumps).
- Sticky headers, grouped rows, and nested lists require custom implementations.
- Windowing libraries do not virtualise horizontal grids by default; use `FixedSizeGrid` or TanStack Virtual's grid support.
- Print and export (Ctrl+P, CSV export) must account for the fact that only visible items exist in the DOM.

### Annotated Code Example: Virtualised Contact List

```jsx
import { useRef, useState, memo } from 'react';
import { useVirtualizer } from '@tanstack/react-virtual';

const ContactRow = memo(function ContactRow({ contact, style }) {
  return (
    <div
      style={{
        ...style,
        display: 'flex',
        alignItems: 'center',
        padding: '0 16px',
        borderBottom: '1px solid #eee',
      }}
    >
      <img
        src={contact.avatar}
        alt=""
        width={32}
        height={32}
        style={{ borderRadius: '50%', marginRight: 12 }}
      />
      <div>
        <p style={{ margin: 0, fontWeight: 600 }}>{contact.name}</p>
        <p style={{ margin: 0, color: '#666', fontSize: 14 }}>{contact.email}</p>
      </div>
    </div>
  );
});

export default function ContactList({ contacts }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: contacts.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 56,
    overscan: 10,
  });

  return (
    <div
      ref={parentRef}
      style={{ height: 600, overflow: 'auto', border: '1px solid #ddd', borderRadius: 8 }}
    >
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <ContactRow
            key={virtualItem.key}
            contact={contacts[virtualItem.index]}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: virtualItem.size,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          />
        ))}
      </div>
    </div>
  );
}
```

**Expected Output:** A scrollable contact list with 10,000 contacts. Only ~20 rows are in the DOM at any time. Scrolling is smooth at 60fps, and memory usage stays constant. The scrollbar reflects the full list height (600,000px for 10,000 items at 56px each).

**Why This Output Occurs:** `useVirtualizer` calculates which items are visible based on the scroll position and the estimated item height. Only those items (plus 10 overscan) are rendered. As the user scrolls, `getVirtualItems()` returns a new range, and the same `ContactRow` components are re-rendered with new data. The total DOM node count stays around 30 regardless of list size.

### Real-World Cases

- **CRM contact lists:** 50,000+ contacts with avatars and metadata.
- **E-commerce product grids:** 10,000+ products with images and prices.
- **Log viewers:** 100,000+ log lines with filtering and search.
- **Data tables:** Financial or analytics tables with thousands of rows.
- **Chat histories:** 10,000+ messages with variable heights.
- **Virtualised grids:** Image galleries, spreadsheet-like editors, and calendar views.

### References

- react-window: https://github.com/bvaughn/react-window
- react-virtualized: https://github.com/bvaughn/react-virtualized
- TanStack Virtual: https://tanstack.com/virtual/latest
- web.dev – Virtualize Large Lists with react-window: https://web.dev/articles/virtualize-long-lists-react-window
- MDN Web Docs – ResizeObserver: https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver
- Addy Osmani – Rendering Large Lists with React Virtualised: https://addyosmani.com/blog/react-window/

---

## Core Concept 2: State Partitioning & Colocation

### Definitions

**Core Definition:** State partitioning is the practice of splitting a monolithic global store into smaller, domain-scoped stores, while colocation is the practice of moving state down to the component that actually uses it—together, they minimise the blast radius of every state update.

**Technical Definition:** In a large application, a single global store (Redux, Context, Zustand) causes "dispatch storms": a single update re-renders every component subscribed to that store, even if only a small slice changed. State partitioning addresses this by (1) **splitting the store by domain** (e.g., `userStore`, `cartStore`, `uiStore`) so updates to one domain do not affect components in another; (2) **scoping state to features** (Feature-Sliced Design, module boundaries) so that cross-feature updates are explicit; and (3) **using selector-based subscriptions** (`useSelector`, `useStore(selector)`) so components only re-render when their selected slice changes. Colocation addresses the same problem from below: move state into the smallest component that reads and writes it, so updates never propagate to ancestors or siblings. The complementary principle is "lift only when necessary": state should live at the lowest common ancestor of all components that need it, not at the root.

**Beginner-Friendly Explanation:** Imagine a company-wide announcement system where every announcement goes to every employee, even if it is only relevant to the accounting department. That is a global store. State partitioning is like having department-specific mailing lists: the accounting announcement goes only to accounting. Colocation is like having a private conversation with the person next to you instead of announcing it to the whole company. Both reduce noise and improve performance.

### Purposes

- To prevent a single state update from re-rendering the entire application.
- To isolate failures and changes to the feature or domain that owns them.
- To reduce the number of components subscribed to any single store.
- To enable independent development and testing of features.
- To make the state architecture explicit and maintainable.
- To reduce memory pressure by not holding unused state at the root.

### Syntax Rules and Structure

**Partitioning a Zustand Store by Domain:**
```javascript
// ❌ Bad: one giant store
const useAppStore = create((set) => ({
  user: null,
  cart: [],
  theme: 'light',
  notifications: [],
  setUser: (user) => set({ user }),
  addToCart: (item) => set((s) => ({ cart: [...s.cart, item] })),
  setTheme: (theme) => set({ theme }),
}));

// ✅ Good: separate stores per domain
const useUserStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));

const useCartStore = create((set) => ({
  cart: [],
  addToCart: (item) => set((s) => ({ cart: [...s.cart, item] })),
}));

const useUIStore = create((set) => ({
  theme: 'light',
  setTheme: (theme) => set({ theme }),
}));
```

**Component Breakdown:**
- Each store owns one domain.
- Components subscribe only to the stores they need.
- Updating `cart` does not re-render components that only read `user`.

**Selector-Based Subscriptions:**
```jsx
// ❌ Bad: subscribes to the entire store
const { user, cart, theme } = useAppStore();
// Re-renders on any change to any slice

// ✅ Good: subscribes to specific slices
const user = useUserStore((s) => s.user);
const cartCount = useCartStore((s) => s.cart.length);
// Re-renders only when the selected slice changes
```

**Component Breakdown:**
- `useUserStore((s) => s.user)`: The selector returns only `user`.
- Zustand uses `Object.is` to compare the selector result; the component re-renders only if the selected value changes.
- For objects, use `useShallow` to prevent re-renders when the object contents are equal.

**Colocation: Moving State Down**
```jsx
// ❌ Bad: state in the parent re-renders the heavy sibling
function Page() {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setIsOpen(o => !o)}>Toggle</button>
      {isOpen && <Panel />}
      <HeavyChart />
    </div>
  );
}

// ✅ Good: state colocated in a small component
function Page() {
  return (
    <div>
      <TogglePanel />
      <HeavyChart />
    </div>
  );
}

function TogglePanel() {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setIsOpen(o => !o)}>Toggle</button>
      {isOpen && <Panel />}
    </div>
  );
}
```

**Component Breakdown:**
- Bad version: `isOpen` lives in `Page`; toggling re-renders `Page` and all its children, including `HeavyChart`.
- Good version: `isOpen` lives in `TogglePanel`; toggling re-renders only `TogglePanel`. `HeavyChart` never re-renders.

**Splitting Context by Concern:**
```jsx
// ❌ Bad: one context with state and actions
const AppContext = createContext(null);

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const value = useMemo(() => ({ user, setUser, theme, setTheme }), [user, theme]);
  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// ✅ Good: separate contexts for state and actions, and per concern
const UserContext = createContext(null);
const UserActionsContext = createContext(null);
const ThemeContext = createContext(null);
const ThemeActionsContext = createContext(null);
```

**Component Breakdown:**
- State-only consumers subscribe to `UserContext` and re-render only when `user` changes.
- Action-only consumers subscribe to `UserActionsContext` (stable reference) and never re-render.
- Theme consumers are independent of user consumers.

**Syntax Rules:**
- Partition stores by domain (user, cart, UI, notifications, etc.).
- Use selector-based subscriptions to subscribe only to the slices a component needs.
- Use `useShallow` (Zustand) for object/array selectors to prevent re-renders when contents are equal.
- Colocate state in the smallest component that reads and writes it.
- Lift state only to the closest common ancestor of all components that need it.
- Split Context into separate state and dispatch contexts, and per concern.
- Memoise context values with `useMemo` to prevent consumer re-renders when the provider re-renders.
- Use feature-based boundaries (Feature-Sliced Design) to scope state to a feature.

**Constraints and Limitations:**
- Partitioning adds boilerplate; a single store is simpler for small apps.
- Cross-domain updates (e.g., logout clears cart and user) require explicit coordination.
- Colocation can lead to prop drilling if not combined with slots or context.
- Over-partitioning (a store per component) defeats the purpose; group by domain, not by component.
- Selector functions must be stable or memoised; inline selectors can cause re-renders.

### Annotated Code Example: Partitioned Stores with Selectors

```jsx
import { create } from 'zustand';
import { useShallow } from 'zustand/react/shallow';

// --- User store ---
const useUserStore = create((set) => ({
  user: null,
  login: (user) => set({ user }),
  logout: () => set({ user: null }),
}));

// --- Cart store ---
const useCartStore = create((set) => ({
  items: [],
  addItem: (item) => set((s) => ({ items: [...s.items, item] })),
  removeItem: (id) => set((s) => ({ items: s.items.filter(i => i.id !== id) })),
  clear: () => set({ items: [] }),
}));

// --- Component that only reads the user ---
function UserBadge() {
  const user = useUserStore((s) => s.user);
  console.log('UserBadge render');
  return <span>{user?.name ?? 'Guest'}</span>;
}

// --- Component that only reads the cart count ---
function CartBadge() {
  const count = useCartStore((s) => s.items.length);
  console.log('CartBadge render');
  return <span>Cart: {count}</span>;
}

// --- Component that reads multiple slices (with shallow comparison) ---
function CartSummary() {
  const { items, removeItem } = useCartStore(
    useShallow((s) => ({ items: s.items, removeItem: s.removeItem }))
  );
  console.log('CartSummary render');
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </li>
      ))}
    </ul>
  );
}

export default function App() {
  const addItem = useCartStore((s) => s.addItem);

  return (
    <div>
      <UserBadge />
      <CartBadge />
      <CartSummary />
      <button onClick={() => addItem({ id: Date.now(), name: 'New Item' })}>
        Add to Cart
      </button>
    </div>
  );
}
```

**Expected Output (console):**
```
UserBadge render
CartBadge render
CartSummary render
--- clicking "Add to Cart" ---
CartBadge render
CartSummary render
(no UserBadge render)
```

**Why This Output Occurs:** `UserBadge` subscribes only to `useUserStore`; adding an item to the cart does not affect it. `CartBadge` subscribes to `items.length`, so it re-renders when the count changes. `CartSummary` subscribes to `items` and `removeItem` with `useShallow`, so it re-renders when the items array changes. The partitioned stores ensure that updates to one domain do not affect components in another.

### Real-World Cases

- **E-commerce:** Separate stores for user, cart, products, and UI preferences.
- **SaaS dashboards:** Separate stores per feature (analytics, billing, settings).
- **Multi-tenant apps:** Separate stores per tenant or workspace.
- **Notifications:** A dedicated notification store that only notification components subscribe to.
- **Theming:** A theme store that only theme-aware components subscribe to.

### References

- Zustand – Comparison: https://zustand.docs.pmnd.rs/learn/getting-started/comparison
- Zustand – useShallow: https://zustand.docs.pmnd.rs/hooks/use-shallow
- Redux – Code Structure: https://redux.js.org/usage/structuring-reducers/structuring-reducers
- React Official Documentation – Choosing the State Structure: https://react.dev/learn/choosing-the-state-structure
- React Official Documentation – Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context
- Feature-Sliced Design: https://feature-sliced.design/
- Kent C. Dodds – One React Mistake That's Keeping Your App Slow: https://kentcdodds.com/blog/optimize-react-re-renders

---

## Core Concept 3: Concurrent Transitions

### Definitions

**Core Definition:** Concurrent transitions are React 18+ features (`useTransition`, `useDeferredValue`) that let you mark non-urgent updates as interruptible, keeping the UI responsive (typing, scrolling) while a heavy render runs in the background.

**Technical Definition:** `useTransition()` returns `[isPending, startTransition]`. Wrapping a state update in `startTransition` marks it as non-urgent: React renders it in the background and interrupts it if a more urgent update (a keystroke, a click) occurs. `useDeferredValue(value)` returns a deferred version of a value that lags behind the original; components using the deferred value re-render at a lower priority. Both rely on React's concurrent renderer, which can pause, prioritise, and resume rendering work. `useTransition` is for the code that *triggers* the update (you control the setter); `useDeferredValue` is for the code that *receives* the value (you do not control the update). In React 19, `startTransition` supports async functions (Actions), enabling pending states for async work.

**Beginner-Friendly Explanation:** Imagine you are typing in a search box while a huge list of results is rendering. Without transitions, the list update blocks the typing, and the input feels laggy. With transitions, React says: "Typing is urgent; rendering the list can wait." If you keep typing, React interrupts the list rendering and restarts it with the new query. The input stays responsive, and the list catches up when you pause.

### Purposes

- To keep typing, scrolling, and clicking responsive during heavy renders.
- To mark non-urgent updates (filtering, tab switching, navigation) as transitions.
- To show a pending indicator while a transition is in progress.
- To defer expensive renders of values you do not control.
- To interrupt long renders when the user interacts.

### Syntax Rules and Structure

**useTransition (You Control the Setter):**
```jsx
import { useState, useTransition } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    const value = e.target.value;
    setQuery(value); // Urgent: update the input immediately

    startTransition(() => {
      setResults(filterLargeList(value)); // Non-urgent: update results
    });
  }

  return (
    <div>
      <input value={query} onChange={handleChange} placeholder="Search..." />
      {isPending && <span>Updating results...</span>}
      <ul>
        {results.map((r) => <li key={r.id}>{r.name}</li>)}
      </ul>
    </div>
  );
}
```

**Component Breakdown:**
- `setQuery(value)`: Urgent update (input value).
- `startTransition(() => setResults(...))`: Non-urgent update (results list).
- `isPending`: `true` while the transition is running.
- React interrupts the transition if the user types again.

**useDeferredValue (You Do Not Control the Setter):**
```jsx
import { useState, useDeferredValue, memo } from 'react';

const SlowList = memo(function SlowList({ query }) {
  const items = [];
  for (let i = 0; i < 50000; i++) {
    if (String(i).includes(query)) items.push(i);
  }
  return <ul>{items.slice(0, 100).map((i) => <li key={i}>{i}</li>)}</ul>;
});

export default function FilterApp() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Filter numbers..."
      />
      <p>{isStale ? 'Updating...' : 'Results'}</p>
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <SlowList query={deferredQuery} />
      </div>
    </div>
  );
}
```

**Component Breakdown:**
- `deferredQuery = useDeferredValue(query)`: A deferred version of `query`.
- The input uses `query` (urgent); `SlowList` uses `deferredQuery` (non-urgent).
- `isStale`: `true` when the deferred value lags behind.
- The list is dimmed while stale.

**React 19: Async Transition (Actions):**
```jsx
import { useState, useTransition } from 'react';

function SaveButton() {
  const [isPending, startTransition] = useTransition();
  const [status, setStatus] = useState('idle');

  function handleSave() {
    startTransition(async () => {
      setStatus('saving');
      await save();
      setStatus('saved');
    });
  }

  return (
    <button onClick={handleSave} disabled={isPending}>
      {isPending ? 'Saving...' : 'Save'}
    </button>
  );
}
```

**Component Breakdown:**
- `startTransition(async () => { ... })`: React 19 supports async callbacks.
- `isPending`: `true` until the async work completes.
- The UI stays responsive during the async operation.

**Syntax Rules:**
- Use `useTransition` when you control the state setter and want to mark specific updates as non-urgent.
- Use `useDeferredValue` when you receive a value (prop) and want to defer its expensive render.
- Keep urgent updates (input value, click feedback) outside the transition.
- Use `isPending` (or compare `value !== deferredValue`) to show a stale/pending indicator.
- Memoise the slow component with `React.memo` so it re-renders only when the deferred value changes.
- In React 18, `startTransition` callbacks must be synchronous; use React 19 for async.
- Do not use transitions for controlled input values; the input must update urgently.

**Constraints and Limitations:**
- Transitions do not prevent the state update; they only make it interruptible.
- `isPending` does not tell you which transition is running when there are multiple.
- `useDeferredValue` only helps if the slow render is actually expensive; otherwise, it adds overhead.
- Transitions do not work with state updates outside `startTransition`.
- Concurrent rendering requires React 18+; React 17 and earlier render synchronously.
- Transitions do not help with network latency; they only help with render time.

### Annotated Code Example: Tab Switching with Transitions

```jsx
import { useState, useTransition, memo } from 'react';

const SlowTab = memo(function SlowTab({ query }) {
  const items = [];
  for (let i = 0; i < 10000; i++) {
    if (String(i).includes(query)) items.push(i);
  }
  return <ul>{items.map((i) => <li key={i}>{i}</li>)}</ul>;
});

export default function TabSwitcher() {
  const [tab, setTab] = useState('home');
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  function switchTab(newTab) {
    startTransition(() => {
      setTab(newTab);
    });
  }

  return (
    <div>
      <div>
        <button onClick={() => switchTab('home')} disabled={tab === 'home'}>
          Home
        </button>
        <button onClick={() => switchTab('list')} disabled={tab === 'list'}>
          List
        </button>
        {isPending && <span>Loading...</span>}
      </div>

      {tab === 'home' && <p>Welcome to the home tab.</p>}
      {tab === 'list' && (
        <div>
          <input
            value={query}
            onChange={(e) => setQuery(e.target.value)}
            placeholder="Filter numbers..."
          />
          <SlowTab query={query} />
        </div>
      )}
    </div>
  );
}
```

**Expected Output:** Clicking "List" switches to a tab with 10,000 items. The tab switch is marked as a transition, so the UI shows "Loading..." while the slow list renders. If the user clicks "Home" while the list is rendering, React interrupts the list render and switches back immediately.

**Why This Output Occurs:** `startTransition(() => setTab(newTab))` marks the tab change as non-urgent. React begins rendering the slow list but keeps the UI responsive. `isPending` is `true` while the transition is running, showing "Loading...". If the user clicks another tab, React interrupts the current transition and processes the new one.

### Real-World Cases

- **Search boxes:** Filtering large lists while keeping typing responsive.
- **Tab switching:** Switching between heavy tabs without blocking.
- **Data table filtering:** Filtering and sorting large tables without input lag.
- **Dashboard widgets:** Re-rendering heavy widgets without blocking the UI.
- **Theme switching:** Re-rendering the whole app with a new theme without blocking.
- **Route transitions:** Navigating to a new route while keeping the current page interactive.

### References

- React Official Documentation – `useTransition`: https://react.dev/reference/react/useTransition
- React Official Documentation – `useDeferredValue`: https://react.dev/reference/react/useDeferredValue
- React Official Documentation – `startTransition`: https://react.dev/reference/react/startTransition
- React Official Documentation – Concurrent Rendering: https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react
- React 19 – Actions: https://react.dev/reference/react/useTransition#performing-actions-in-transitions
- web.dev – Keeping the Main Thread Free with Transitions: https://web.dev/articles/optimize-long-tasks

---

## Core Concept 4: Asynchronous & Network Strategy

### Definitions

**Core Definition:** Asynchronous and network strategy is the set of techniques—debouncing, throttling, streaming HTTP, data-fetching caches, and GraphQL payload minimisation—that reduce the frequency, size, and cost of network requests in a large application.

**Technical Definition:** Network strategy operates at four layers. **Input debouncing and throttling** reduce the number of requests triggered by user input (search, resize, scroll). **Data-fetching caches** (TanStack Query, SWR) deduplicate concurrent requests, serve stale data while revalidating, and persist data across navigations. **Streaming HTTP architectures** (HTTP/2, Server-Sent Events, streaming SSR) allow the server to push data incrementally, avoiding a single large response. **GraphQL payload minimisation** uses field selection to fetch only the fields the UI needs, and persisted queries to reduce request size. Together, these techniques address the most common network bottlenecks: too many requests, requests that are too large, and requests that block the UI while waiting for a response.

**Beginner-Friendly Explanation:** Network requests are often the slowest part of a web app. If you send a request on every keystroke, you overwhelm the server and slow down the UI. If you fetch data you do not need, you waste bandwidth. If you wait for one big response before rendering anything, the user stares at a blank screen. Network strategy is about asking for less, less often, and rendering what you get as soon as it arrives.

### Purposes

- To reduce the number of network requests triggered by user input.
- To deduplicate concurrent requests and cache responses.
- To stream data incrementally rather than blocking on a single response.
- To fetch only the fields the UI needs (GraphQL).
- To reduce request and response payload sizes.
- To keep the UI responsive while network requests are in flight.

### Syntax Rules and Structure

**Debouncing Input (Custom Hook):**
```jsx
import { useState, useEffect } from 'react';

function useDebouncedValue(value, delay = 300) {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
}

function SearchBox() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebouncedValue(query, 300);

  useEffect(() => {
    if (debouncedQuery) {
      fetchResults(debouncedQuery);
    }
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

**Component Breakdown:**
- `useDebouncedValue`: Returns a value that updates only after `delay` ms of inactivity.
- The fetch runs only when the debounced value changes, not on every keystroke.

**Throttling Scroll/Resize:**
```jsx
function useThrottledValue(value, interval = 100) {
  const [throttled, setThrottled] = useState(value);
  const lastRun = useRef(Date.now());

  useEffect(() => {
    const now = Date.now();
    if (now - lastRun.current >= interval) {
      lastRun.current = now;
      setThrottled(value);
    } else {
      const timer = setTimeout(() => {
        lastRun.current = Date.now();
        setThrottled(value);
      }, interval - (now - lastRun.current));
      return () => clearTimeout(timer);
    }
  }, [value, interval]);

  return throttled;
}
```

**Component Breakdown:**
- Throttling ensures the value updates at most once per `interval` ms.
- Useful for scroll position, mouse position, and resize events.

**TanStack Query (Deduplication + Cache):**
```jsx
import { useQuery } from '@tanstack/react-query';

function SearchResults({ query }) {
  const { data, isPending } = useQuery({
    queryKey: ['search', query],
    queryFn: () => fetchResults(query),
    staleTime: 60_000,        // Fresh for 1 minute
    enabled: query.length > 2, // Only run when query is long enough
    placeholderData: keepPreviousData, // Keep previous results while loading
  });

  if (isPending) return <p>Loading...</p>;
  return <ul>{data.map((r) => <li key={r.id}>{r.title}</li>)}</ul>;
}
```

**Component Breakdown:**
- `queryKey`: Unique key for caching and deduplication.
- `staleTime`: How long data is considered fresh.
- `enabled`: Gating the query on a condition.
- `placeholderData`: Keep previous data while loading new data.

**Streaming SSR (Next.js App Router):**
```tsx
// app/page.tsx
import { Suspense } from 'react';
import { SlowComponent } from './SlowComponent';

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<Skeleton />}>
        <SlowComponent />   {/* Streams in when ready */}
      </Suspense>
    </div>
  );
}
```

**Component Breakdown:**
- The shell (heading) renders immediately.
- `SlowComponent` streams in when its data resolves.
- The user sees the page structure immediately, not a blank screen.

**GraphQL Field Selection:**
```graphql
# ❌ Bad: over-fetching
query GetUser {
  user {
    id
    name
    email
    address
    phone
    orders { id total items { id name price } }
  }
}

# ✅ Good: only what the UI needs
query GetUser {
  user {
    id
    name
    email
  }
}
```

**Component Breakdown:**
- Only request the fields the current view uses.
- Use fragments to share field selections across queries.
- Use persisted queries to reduce request size.

**Syntax Rules:**
- Debounce input-triggered requests by 300–500ms.
- Throttle scroll/resize handlers by 100–200ms.
- Use a data-fetching cache (TanStack Query, SWR) for all server state.
- Set `staleTime` based on how often the data changes.
- Use `enabled` to gate queries on conditions.
- Use `placeholderData: keepPreviousData` for pagination.
- Stream SSR content with `<Suspense>` so the shell renders immediately.
- In GraphQL, request only the fields the UI needs; use fragments and persisted queries.

**Constraints and Limitations:**
- Debouncing adds latency to the user's input; 300ms is usually imperceptible, but 1000ms is not.
- Throttling can miss the final value; pair with a trailing update.
- Caching requires cache invalidation; stale data can mislead users.
- Streaming SSR requires framework support (Next.js, Remix).
- GraphQL field selection requires discipline; over-fetching creeps back in without review.
- Persisted queries require build-time tooling and server support.

### Annotated Code Example: Debounced Search with TanStack Query

```jsx
import { useState, useEffect } from 'react';
import { useQuery, keepPreviousData } from '@tanstack/react-query';

function useDebouncedValue(value, delay = 300) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}

async function searchProducts(query) {
  const res = await fetch(`/api/products?q=${encodeURIComponent(query)}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

export default function ProductSearch() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebouncedValue(query, 300);

  const { data, isPending, isFetching } = useQuery({
    queryKey: ['products', debouncedQuery],
    queryFn: () => searchProducts(debouncedQuery),
    enabled: debouncedQuery.length > 2,
    staleTime: 60_000,
    placeholderData: keepPreviousData,
  });

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search products..."
      />
      {isPending && <p>Loading...</p>}
      {isFetching && !isPending && <p>Updating...</p>}
      <ul>
        {data?.map((product) => <li key={product.id}>{product.name}</li>)}
      </ul>
    </div>
  );
}
```

**Expected Output:** Typing in the input updates the value immediately. After 300ms of inactivity, the query fires. While the query is loading, previous results remain visible (via `placeholderData`). The input stays responsive because the fetch is debounced.

**Why This Output Occurs:** `useDebouncedValue` delays the query update by 300ms. TanStack Query caches results by `queryKey`, deduplicates concurrent requests, and keeps previous data while loading new data. The input value is urgent and updates immediately; the query is non-urgent and follows the debounced value.

### Real-World Cases

- **Search autocomplete:** Debounced queries with cached results.
- **Infinite scroll:** Cursor-based pagination with `useInfiniteQuery`.
- **Real-time dashboards:** Streaming SSR or WebSocket updates with cache write-through.
- **E-commerce filters:** Debounced filter changes with URL state and cached results.
- **Analytics:** GraphQL queries with field selection and persisted queries.
- **Collaborative editors:** WebSocket or SSE for real-time updates.

### References

- TanStack Query – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query – Caching: https://tanstack.com/query/latest/docs/framework/react/guides/caching
- TanStack Query – Paginated Queries: https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries
- TanStack Query – Query Cancellation: https://tanstack.com/query/latest/docs/framework/react/guides/query-cancellation
- MDN Web Docs – Debouncing and Throttling: https://developer.mozilla.org/en-US/docs/Web/API/Document/scroll_event
- web.dev – Streaming SSR: https://web.dev/articles/rendering-on-the-web
- GraphQL – Field Selection: https://graphql.org/learn/queries/
- Apollo – Persisted Queries: https://www.apollographql.com/docs/apollo-server/performance/apq/
- web.dev – Reduce JavaScript Payloads: https://web.dev/articles/reduce-javascript-payloads-with-code-splitting

---

## Core Concept 5: Asset & Bundle Engineering

### Definitions

**Core Definition:** Asset and bundle engineering is the practice of reducing the size and number of JavaScript, CSS, image, and font assets shipped to the browser—using bundle analysers, tree-shaking, code splitting, and asset optimisation—so that the application loads quickly on any device.

**Technical Definition:** Asset and bundle engineering operates on four fronts. **Bundle analysis** uses tools (`webpack-bundle-analyzer`, `vite-bundle-visualizer`, `source-map-explorer`) to visualise what is in the bundle and identify bloat. **Tree-shaking** eliminates unused exports from ES modules, but it only works if modules are side-effect-free and imports are static. **Code splitting** (covered in a separate cheat sheet) divides the bundle into chunks loaded on demand. **Asset optimisation** includes image compression (WebP, AVIF), responsive images (`srcset`, `sizes`), font subsetting (`unicode-range`, `font-display: swap`), and CSS optimisation (purging unused CSS, minification). The goal is to ship the smallest possible payload on the critical path, defer everything else, and cache aggressively.

**Beginner-Friendly Explanation:** Your app's JavaScript, CSS, images, and fonts are all downloaded by the browser. If they are large, the app is slow—especially on mobile networks. Asset and bundle engineering is about making those files smaller, splitting them into pieces so the browser only downloads what it needs, and caching them so repeat visits are fast. Bundle analysers show you what is taking up space; tree-shaking removes dead code; image and font optimisation shrink the biggest assets.

### Purposes

- To identify and eliminate bloat in the JavaScript bundle.
- To remove dead code via tree-shaking.
- To reduce the initial payload with code splitting.
- To optimise images (compression, format, responsive sizes).
- To optimise fonts (subsetting, `font-display`, preloading).
- To purge unused CSS and minify all assets.
- To cache assets aggressively with content hashing.

### Syntax Rules and Structure

**Bundle Analysis (Vite):**
```bash
npm install -D rollup-plugin-visualizer
```

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({ filename: 'stats.html', gzipSize: true, brotliSize: true }),
  ],
});
```

**Component Breakdown:**
- `rollup-plugin-visualizer`: Generates an interactive treemap of the bundle.
- `gzipSize` / `brotliSize`: Shows compressed sizes.
- Open `stats.html` after building to inspect the bundle.

**Bundle Analysis (Webpack):**
```bash
npm install -D webpack-bundle-analyzer
```

```javascript
// webpack.config.js
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
  plugins: [new BundleAnalyzerPlugin()],
};
```

**Tree-Shaking-Friendly Imports:**
```javascript
// ❌ Bad: imports the entire library
import _ from 'lodash';
_.debounce(fn, 300);

// ✅ Good: imports only the function (tree-shakeable)
import debounce from 'lodash/debounce';
debounce(fn, 300);

// ✅ Best: use the ESM build
import { debounce } from 'lodash-es';
```

**Component Breakdown:**
- `import _ from 'lodash'`: Imports the entire library (hundreds of KB).
- `import debounce from 'lodash/debounce'`: Imports only the `debounce` module.
- `lodash-es`: The ESM build, which is fully tree-shakeable.

**Image Optimisation:**
```html
<!-- ❌ Bad: one large image for all viewports -->
<img src="/hero.jpg" alt="Hero" />

<!-- ✅ Good: responsive images with modern formats -->
<picture>
  <source srcset="/hero.avif" type="image/avif" />
  <source srcset="/hero.webp" type="image/webp" />
  <img
    src="/hero.jpg"
    alt="Hero"
    width="1200"
    height="600"
    loading="lazy"
    decoding="async"
    srcset="/hero-400.jpg 400w, /hero-800.jpg 800w, /hero-1200.jpg 1200w"
    sizes="(max-width: 600px) 400px, (max-width: 1200px) 800px, 1200px"
  />
</picture>
```

**Component Breakdown:**
- `<picture>`: Serves AVIF or WebP when supported, falling back to JPEG.
- `srcset` / `sizes`: Serves the appropriate image size for the viewport.
- `loading="lazy"`: Defers off-screen images.
- `decoding="async"`: Does not block rendering.
- `width` / `height`: Prevents layout shift.

**Font Optimisation:**
```css
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-subset.woff2') format('woff2');
  font-display: swap;
  unicode-range: U+0000-00FF, U+0131, U+0152-0153; /* Latin subset */
}

/* Preload the font in the HTML head */
```

```html
<link
  rel="preload"
  href="/fonts/inter-subset.woff2"
  as="font"
  type="font/woff2"
  crossorigin
/>
```

**Component Breakdown:**
- `woff2`: The most compressed font format.
- `unicode-range`: Only download the glyphs needed for the language.
- `font-display: swap`: Show fallback text immediately, swap when the font loads.
- `<link rel="preload">`: Start the font download early.

**CSS Purging (Tailwind):**
```javascript
// tailwind.config.js
module.exports = {
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  // Tailwind v4: automatic content detection; no config needed
};
```

**Component Breakdown:**
- Tailwind scans the listed files and generates only the utility classes used.
- Unused CSS is never emitted, keeping the stylesheet minimal.

**Syntax Rules:**
- Run a bundle analyser before and after optimisation to measure impact.
- Use ESM builds and named imports for tree-shaking.
- Avoid importing entire libraries (`lodash`, `moment`, `rxjs`).
- Use `loading="lazy"` and `decoding="async"` for off-screen images.
- Serve modern image formats (AVIF, WebP) with fallbacks.
- Use responsive images (`srcset`, `sizes`) to avoid downloading oversized images.
- Subset fonts and use `font-display: swap`.
- Preload critical fonts and images.
- Purge unused CSS (Tailwind does this automatically).
- Content-hash asset filenames for aggressive caching.

**Constraints and Limitations:**
- Tree-shaking requires ES modules; CommonJS modules are not tree-shakeable.
- Some libraries have side effects that prevent tree-shaking; check the library's `sideEffects` field.
- Image optimisation requires a build pipeline (Sharp, Squoosh, ImageOptim).
- Font subsetting requires a tool (glyphhanger, Fonttools).
- AVIF has limited browser support in older browsers; always provide a fallback.
- Over-optimising images can degrade visual quality; test on real devices.

### Annotated Code Example: Optimised Bundle and Assets

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({ filename: 'stats.html', gzipSize: true, brotliSize: true }),
  ],
  build: {
    target: 'es2020',
    minify: 'esbuild',
    cssMinify: true,
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-chart': ['chart.js', 'react-chartjs-2'],
          'vendor-editor': ['@codemirror/state', '@codemirror/view'],
        },
      },
    },
  },
});
```

```jsx
// Image component with modern formats and lazy loading
function OptimisedImage({ src, alt, width, height }) {
  return (
    <picture>
      <source srcSet={`${src}.avif`} type="image/avif" />
      <source srcSet={`${src}.webp`} type="image/webp" />
      <img
        src={`${src}.jpg`}
        alt={alt}
        width={width}
        height={height}
        loading="lazy"
        decoding="async"
        style={{ maxWidth: '100%', height: 'auto' }}
      />
    </picture>
  );
}
```

**Expected Output:** The bundle is split into vendor chunks (React, Chart.js, CodeMirror), each cached separately. Images are served as AVIF or WebP when supported, with a JPEG fallback. Off-screen images are lazy-loaded. The bundle analyser shows the size of each chunk and dependency.

**Why This Output Occurs:** `manualChunks` groups heavy dependencies into named chunks, so React is not re-downloaded when Chart.js changes. The `<picture>` element serves the best format the browser supports. `loading="lazy"` defers off-screen images. The bundle analyser visualises the result, showing whether the optimisation was effective.

### Real-World Cases

- **E-commerce:** Optimised product images (WebP, responsive sizes), tree-shaken UI libraries, split vendor chunks.
- **Marketing sites:** Font subsetting, preloaded hero images, minimal JavaScript.
- **SaaS dashboards:** Split chart/editor chunks, lazy-loaded widgets, purged CSS.
- **Documentation:** Code-split routes, optimised code sample highlighting, cached assets.
- **Mobile web:** Aggressive image optimisation, font subsetting, minimal critical path.

### References

- web.dev – Reduce JavaScript Payloads: https://web.dev/articles/reduce-javascript-payloads-with-code-splitting
- web.dev – Optimise Images: https://web.dev/learn/images/
- web.dev – Font Best Practices: https://web.dev/articles/font-best-practices
- Webpack – Tree Shaking: https://webpack.js.org/guides/tree-shaking/
- Webpack – Bundle Analyzer: https://github.com/webpack-contrib/webpack-bundle-analyzer
- Vite – Build Optimisation: https://vitejs.dev/guide/build.html
- Rollup – Plugin Visualizer: https://github.com/btd/rollup-plugin-visualizer
- MDN Web Docs – Responsive Images: https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images
- MDN Web Docs – `<picture>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture
- MDN Web Docs – `font-display`: https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display
- Squoosh: https://squoosh.app/
- Glyphhanger: https://github.com/filamentgroup/glyphhanger

---

## Comparison and Decision Guidance

| Pillar | Primary Tool/Technique | When to Use | Key Risk |
|---|---|---|---|
| **Windowing** | `react-window`, TanStack Virtual | Lists/grids with 100+ items | Fixed-height requirement; accessibility gaps |
| **State Partitioning** | Zustand stores, selectors, Context splitting | Global state causing wide re-renders | Boilerplate; cross-domain coordination |
| **Colocation** | Move state to smallest component | State updates causing wide re-renders | Prop drilling if overdone |
| **Concurrent Transitions** | `useTransition`, `useDeferredValue` | Heavy renders blocking typing/scrolling | Does not help network latency |
| **Network Strategy** | Debouncing, TanStack Query, streaming, GraphQL | Frequent or large network requests | Cache invalidation; over-fetching |
| **Bundle Engineering** | Bundle analyser, tree-shaking, code splitting | Large JavaScript bundles | Over-splitting; tree-shaking limitations |
| **Asset Optimisation** | AVIF/WebP, font subsetting, lazy loading | Large images and fonts | Browser support; build complexity |

**Decision Guidance:**
- **Start with measurement:** Use React DevTools Profiler, Lighthouse, and a bundle analyser to identify the actual bottleneck.
- **Windowing is non-negotiable for large lists:** If you render more than 100 items, virtualise.
- **Partition state by domain:** If a single store update re-renders unrelated components, split the store.
- **Colocate state:** If a state update re-renders components that do not use it, move it down.
- **Use transitions for expensive renders:** If typing feels laggy while a list renders, use `useDeferredValue` or `useTransition`.
- **Debounce input-triggered requests:** If every keystroke fires a request, debounce by 300ms.
- **Cache server state:** If the same data is fetched repeatedly, use TanStack Query or SWR.
- **Stream SSR content:** If the initial page load is slow, use Suspense and streaming.
- **Analyse the bundle:** If the initial bundle is large, run a bundle analyser and tree-shake.
- **Optimise images and fonts:** If the page is image- or font-heavy, use modern formats, responsive sizes, and subsetting.
- **Validate every optimisation:** Measure before and after; remove changes that do not help.

---

## References

- React Official Documentation – `useTransition`: https://react.dev/reference/react/useTransition
- React Official Documentation – `useDeferredValue`: https://react.dev/reference/react/useDeferredValue
- React Official Documentation – Choosing the State Structure: https://react.dev/learn/choosing-the-state-structure
- React Official Documentation – Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context
- React Official Documentation – `<Suspense>`: https://react.dev/reference/react/Suspense
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler
- React Official Documentation – Profiler: https://react.dev/reference/react/Profiler
- react-window: https://github.com/bvaughn/react-window
- react-virtualized: https://github.com/bvaughn/react-virtualized
- TanStack Virtual: https://tanstack.com/virtual/latest
- TanStack Query – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query – Caching: https://tanstack.com/query/latest/docs/framework/react/guides/caching
- TanStack Query – Paginated Queries: https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries
- Zustand – Comparison: https://zustand.docs.pmnd.rs/learn/getting-started/comparison
- Zustand – useShallow: https://zustand.docs.pmnd.rs/hooks/use-shallow
- Redux – Code Structure: https://redux.js.org/usage/structuring-reducers/structuring-reducers
- Feature-Sliced Design: https://feature-sliced.design/
- web.dev – Virtualize Large Lists with react-window: https://web.dev/articles/virtualize-long-lists-react-window
- web.dev – Reduce JavaScript Payloads with Code Splitting: https://web.dev/articles/reduce-javascript-payloads-with-code-splitting
- web.dev – Optimise Images: https://web.dev/learn/images/
- web.dev – Font Best Practices: https://web.dev/articles/font-best-practices
- web.dev – Streaming SSR: https://web.dev/articles/rendering-on-the-web
- MDN Web Docs – `requestIdleCallback`: https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback
- MDN Web Docs – Intersection Observer API: https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API
- MDN Web Docs – Responsive Images: https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images
- MDN Web Docs – `<picture>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture
- MDN Web Docs – `font-display`: https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display
- Webpack – Tree Shaking: https://webpack.js.org/guides/tree-shaking/
- Webpack – Bundle Analyzer: https://github.com/webpack-contrib/webpack-bundle-analyzer
- Vite – Build Optimisation: https://vitejs.dev/guide/build.html
- Rollup – Plugin Visualizer: https://github.com/btd/rollup-plugin-visualizer
- GraphQL – Field Selection: https://graphql.org/learn/queries/
- Apollo – Persisted Queries: https://www.apollographql.com/docs/apollo-server/performance/apq/
- Addy Osmani – The Cost of JavaScript: https://medium.com/@addyosmani/the-cost-of-javascript-in-2018-7d8950fbb5d4
- Squoosh: https://squoosh.app/
- Glyphhanger: https://github.com/filamentgroup/glyphhanger